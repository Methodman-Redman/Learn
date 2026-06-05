- gnome-remote-desktopを停止
```
grdctl rdp disable
systemctl --user stop gnome-remote-desktop
systemctl --user disable gnome-remote-desktop
```
- 確認
  - `grdctl status`
- 必要パッケージのインストール
  - `sudo apt install xrdp tigervnc-standalone-server xfce4`
- xfce4をセッションに設定
  - `echo "startxfce4" > ~/.xsession`
- xrdpを起動
  - `sudo systemctl enable --now xrdp`
  - `sudo systemctl status xrdp`
- log out
- `xfreerdp3 /u:'rdp' /p:'rdp' /v:192.168.31.164 +clipboard /drive:shared,/home/kali/Desktop /dynamic-resolution /cert:ignore`  
- burpsuite
  - `https://portswigger.net/burp/downloads`
  - `chmod +x burpsuite_linux_v2026_4_3.sh`
  - `./burpsuite_linux_v2026_4_3.sh`
    - `/home/ubuntu/BurpSuite`
    - `/usr/local/bin`

- 「Firefox」を外部プロキシとして使う
  - Firefoxのプロキシ設定（設定 > ネットワーク設定）で、手動プロキシを 127.0.0.1 / 8080 に設定
  - http://burp にアクセスし、CA証明書をインストール
    - Firefoxのネットワーク設定でプロキシを 127.0.0.1:8080 に設定した状態で、Firefoxを起動
    - アドレスバーに http://burp と入力してアクセス
      - ※もしこのページが表示されない場合は、Burp Suiteが起動しており、Proxy > Proxy settings で "Proxy Listeners" が 127.0.0.1:8080 で有効になっているか確認してください。
      - 画面右上に表示される "CA Certificate" というリンクをクリックして、cacert.der というファイルを保存
  - 証明書のインポート
    - Firefoxのメニュー（右上の三本線アイコン）から 「設定」 (Settings) 
      - 左側のメニューで 「プライバシーとセキュリティ」 (Privacy & Security) を選択
      - 一番下までスクロールし、「証明書」 (Certificates) セクションにある 「証明書を表示」 (View Certificates...) ボタンをクリック
      - 「証明書マネージャー」が開くので、「認証局証明書」 (Authorities) タブを選択
      - 「インポート...」 (Import...) ボタンを押し、先ほど保存した cacert.der を選択
3. インポート時の信頼設定
ファイルを選択すると「証明書の信頼」ダイアログが表示されます。

「この証明書を信頼して、ウェブサイトを識別する」 (Trust this CA to identify websites) にチェックを入れます。

「OK」 をクリックして完了です。
