# Oura OAuth と Stream Deck 表示を公開用の最小構成で接続する

This ExecPlan is a living document. The sections `Progress`, `Surprises & Discoveries`, `Decision Log`, and `Outcomes & Retrospective` must be kept up to date as work proceeds. It follows `/Users/matsubara/.codex/PLANS.md`.

## Purpose / Big Picture

Oura の日次スコアを、家の Stream Deck の一つのキーに表示する。Oura が新規 Personal Access Token の発行を停止したため、個人用の OAuth 2.0 アプリを作り、認可情報を MacBook の Keychain にだけ保存する。公開するページは、Oura 開発者ポータルで必要な Website、Privacy Policy、Terms of Service の三つだけであり、Oura のデータ、token、利用者情報をページへ送信または公開しない。

人が確認できる完成状態は、`https://ummshun.github.io/oura-stream-deck/` が公開され、Oura Developer Portal のアプリが `daily` scope のみで作成され、本人の同意後に Stream Deck の OURA キーが Readiness、Sleep、Activity の当日スコアを表示することとする。

## Progress

- [x] (2026-11-02 16:59 JST) 新規 Personal Access Token が廃止済みで、OAuth 2.0 アプリが必要と確認した。
- [x] (2026-11-02 16:59 JST) `ummshun/oura-stream-deck` が未作成であること、GitHub の `ummshun` 認証が有効であることを確認した。
- [ ] 静的公開ページ、Privacy Policy、Terms of Service を作成し、ローカル検証する。
- [ ] private で初期commitを作り、secret scan 後に public GitHub repository と GitHub Pages へ公開する。
- [ ] Oura Developer Portal のアプリ作成画面へ公開URL、loopback callback、`daily` scope を入力する。
- [ ] 本人の Oura API Agreement 同意と Create Application を承認後に実行する。
- [ ] OAuth authorization-code flow with PKCE を `stream-deck-control` へ実装し、Keychainで token refresh を維持する。
- [ ] MacBook と mini へ配備し、実スコア表示を読み戻す。

## Surprises & Discoveries

- Observation: Oura の Personal Access Token 画面には、新規tokenを発行できずOAuth 2.0へ移行するよう表示される。
  Evidence: `https://cloud.ouraring.com/personal-access-tokens` の表示文。
- Observation: `shunmatsubara.com` はDNSで解決せず、Oura開発者ポータルへ指定できる既存の公開開発者サイトは確認できなかった。
  Evidence: `curl -IL https://shunmatsubara.com` が host resolution error。

## Decision Log

- Decision: `ummshun.github.io/oura-stream-deck/` をアプリ説明・Privacy・Terms専用の静的サイトにする。
  Rationale: 個人用Oura接続の目的に対して最小の公開範囲であり、既存の業務ポートフォリオや未検証のプロフィールを流用しない。
  Date/Author: 2026-11-02 / Codex and Shun Matsubara.
- Decision: OuraデータのOAuth scopeは `daily` のみとする。
  Rationale: Stream Deckへ表示するReadiness、Sleep、Activityの三スコアに必要な最小権限であり、個人情報、心拍、血中酸素、タグ、ワークアウトは不要である。
  Date/Author: 2026-11-02 / Codex.
- Decision: OAuth redirect URIは `http://127.0.0.1:8765/callback` を第一候補にする。
  Rationale: 認可コードをMacBook外へ転送せず、ローカル一回限りの受信サーバーで処理できる。Oura portalがHTTP loopback URIを拒否した場合は、portalが許容する同等のloopback URIを使い、公開callbackは作らない。
  Date/Author: 2026-11-02 / Codex.

## Outcomes & Retrospective

未完了。PATを前提にした既存のStream Deck実装はOAuth実装へ置換が必要であり、OAuth clientを作成する承認と、本人のOura認可が残っている。

## Context and Orientation

`/Users/matsubara/dev/stream-deck-control` は、家のMac miniにつながったStream DeckからMacBookを操作するPythonとNode.jsのプロジェクトである。`actions.py` がMacBookのアクションを実行し、`plugin/plugin.cjs` がキーの表示を更新する。現行の `oura.py` は廃止されたPersonal Access TokenをKeychainから読むため、OAuthに置換する。

新規の `/Users/matsubara/dev/oura-stream-deck` はGitHub Pages専用の静的サイトにする。動的バックエンド、解析、cookie、フォーム、外部スクリプトは置かない。公開URLはWebsiteに `/`、Privacy Policyに `/privacy/`、Termsに `/terms/` を使う。OAuth client ID、client secret、access token、refresh tokenをGitや公開ページへ保存しない。

OAuth authorization-code flowは、本人がOuraの認可画面で `daily` のみを許可すると一度だけ返る短いcodeを、MacBook上のloopback callbackで受ける手順である。PKCEは、認可要求で作ったランダムなverifierとhashを対にし、第三者が認可codeを横取りして使えないようにする方式である。

## Plan of Work

最初にHTMLとCSSだけで三つの公開ページを作る。トップページは目的、データ最小化、連絡先、PrivacyとTermsへのリンクを示す。Privacyは収集データ、保存場所、保持期間、第三者共有なし、削除方法を明記する。Termsは個人用ツールであること、医療助言ではないこと、Ouraサービスの利用規約が優先することを明記する。公開文章は、実装済みのものだけに限定する。

次にローカルHTTP serverで各URLの200と全リンクを検証する。公開前にrepositoryをprivateで初期化し、secret scannerで全tracked fileにtoken、cookie、credentialがないことを確認する。GitHub Pagesの公開は外部公開なので、remote ownerが`ummshun`であることを確認してから行う。公開後にcurlで三URLの200を確認する。

Oura portalでは、アプリ名を `Shun Matsubara Developer — Oura Stream Deck`、descriptionを `Personal Stream Deck display of my daily Oura Readiness, Sleep, and Activity scores.` とする。Website、Privacy、Termsには公開済みのURLを入れる。scopeは `daily` だけを選ぶ。Contact Emailは本人のOuraアカウント用メールを本人入力または表示済みのアカウント情報から選ぶ。API Agreementの同意とCreate Applicationは永続credentialを生成するため、本人の明示承認後にのみクリックする。

作成後に `stream-deck-control/oura.py` へKeychain credential storage、PKCE authorization、token refresh、daily endpoint fetchを実装する。`authorize_oura.py` は一時的な127.0.0.1 listenerを開き、ブラウザをOAuth同意画面に送り、callbackのstateを照合してtokenをKeychainへ書く。`actions.py` はこのOAuth利用へ切り替え、`plugin/plugin.cjs` は既存のOura専用transportのまま表示を更新する。テストはPKCE state照合、Keychainの呼び出し、refresh token rotation、期限切れの更新、`daily`以外を要求しないことを覆う。

## Concrete Steps

作業ディレクトリは `/Users/matsubara/dev/oura-stream-deck` とする。

    python3 -m http.server 8080
    curl -I http://127.0.0.1:8080/
    curl -I http://127.0.0.1:8080/privacy/
    curl -I http://127.0.0.1:8080/terms/

各コマンドは `HTTP/1.0 200 OK` または `HTTP/1.1 200 OK` を返すことを確認する。

OAuth implementation後は `/Users/matsubara/dev/stream-deck-control` で次を実行する。

    /usr/bin/python3 -m unittest discover -s tests
    node --test tests/*.test.cjs
    /usr/bin/python3 build.py

期待結果は全テストpass、buildが15キーのprofileとpluginを出力することとする。token未設定の状態では `oura_status` は機密値を出さず失敗し、Ouraキー押下はWebを開き続ける。認可済み状態では同actionがReadiness、Sleep、Activityの整数スコアを返す。

## Validation and Acceptance

公開サイトはブラウザでトップ、Privacy、Termsを開け、ページ内リンクが404にならないことを確認する。repositoryの公開後にGitHub Pages URLをcurlし、HTTP 200を確認する。

OAuthは本人がOuraの同意画面で `daily` scopeのみを許可し、MacBookの認可コマンドがKeychainにtokenを保存したことをtoken本文を出さず確認する。最後にminiのStream Deck pluginから `oura_status` を呼び、キーの3行タイトルが実スコアになることをログと実機表示で確認する。

## Idempotence and Recovery

静的ページのローカル検証は何度実行しても状態を変えない。GitHub Pagesの再deployは同じcommitを参照する。OAuthの認可を途中で閉じた場合、listenerを終了して `authorize_oura.py` を再実行する。refresh tokenは一回使うと置換されるため、Keychainの更新はatomicにし、失敗時は古いrefresh tokenを保持する。公開ページやOAuth clientを削除する操作は、別途本人に確認する。

## Artifacts and Notes

公開予定URLは次である。

    https://ummshun.github.io/oura-stream-deck/
    https://ummshun.github.io/oura-stream-deck/privacy/
    https://ummshun.github.io/oura-stream-deck/terms/

Oura OAuth callback候補は次である。

    http://127.0.0.1:8765/callback

## Interfaces and Dependencies

`oura-stream-deck` はブラウザとGitHub Pagesだけを使う静的HTML/CSSであり、JavaScript dependencyを持たない。

`stream-deck-control/oura.py` は次の関数を提供する。

    def status() -> dict:
        """Return today's readiness, sleep, and activity integer scores without exposing credentials."""

    def authorize() -> None:
        """Run a local OAuth callback, verify PKCE state, and store rotated tokens in Keychain."""

Keychain service名は `com.shun.stream-deck-control.oura` を維持するが、account名はtoken種別を区別する。すべてのHTTP requestは `User-Agent: ShunStreamDeck/1.0` とBearer access tokenを使う。

Plan updated 2026-11-02 16:59 JST: created after Oura PAT deprecation required a shift to OAuth and a public policy site.
