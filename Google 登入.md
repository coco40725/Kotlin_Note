## Google API 登入

### 1. 取得 client ID
使用 google 登入 api 需要先有一個 client ID：
1. 到 Google Cloud 新增一個專案（或用現有的專案）
2. 選左邊的憑證 -> 建立憑證 -> OAuth 用戶端 ID

<img src="https://static.coderbridge.com/img/vii120/ad32a8c0d07f4f92a59690947527d38e.png">

3. 首次設定的話，會需要先設定 OAuth 同意畫面，點「設定同意畫面」按鈕 -> User Type 選擇外部，之後可以填寫必填欄位就好
<img src="https://static.coderbridge.com/img/vii120/d76eab079d7941079441456b981cc1c3.png">

4. 設定好同意畫面後，預設的狀態會是「測試中」，只有「測試使用者」才有存取權限，所以要記得把自己的 mail 加進去
<img src="https://static.coderbridge.com/img/vii120/cc2d7be81469407384f7cd21ba391f5c.png">

5. 重新回到剛剛的「建立 OAuth 用戶端 ID」畫面，因為我想要用在網頁登入，所以選擇「網頁應用程式」
<img src="https://static.coderbridge.com/img/vii120/91238e5ebbee438c92756c6031042794.png">

6. 我們會先在本機做測試，所以要把 **localhost** 或其他本地測試網址加到「已授權的 JavaScript 來源」，以 localhost 來說，要分別寫「 http://localhost 」和「 http://localhost:{port} 」

7. 送出後就會拿到 client ID 了，例如:
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.apps.googleusercontent.com

### 2. 加上登入功能
其中 credential 是 base64 的 JWT Token，可以用解碼的方式拿到使用者資料，這邊用 <a href="https://stackoverflow.com/questions/63202266/how-to-extract-full-information-of-the-users-using-google-one-tap-signin">stackoverflow</a> 找到的 function 來執行。
另外，在進行登入功能測試時，記得把網址的 ``127.0.0.1`` 改成 ``localhost``，這樣登入授權才會生效。
```html
<html>
  <body>
      <script src="https://accounts.google.com/gsi/client" async defer></script>
      <div id="g_id_onload"
         data-client_id="sssssssssssssssss.apps.googleusercontent.com" 
         data-callback="handleCallback"
         data-auto_prompt="false">
      </div>
      <div class="g_id_signin"
         data-type="standard"
         data-size="large"
         data-theme="outline"
         data-text="sign_in_with"
         data-shape="rectangular"
         data-logo_alignment="left">
      </div>

      <script>
        // 用於解碼 credential
       function parseJwt (token) {
            var base64Url = token.split('.')[1];
            var base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/');
            var jsonPayload = decodeURIComponent(atob(base64).split('').map(function(c) {
                return '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2);
            }).join(''));

        return JSON.parse(jsonPayload);
        };
        
        function handleCallback(response) {
            const data = parseJwt(response.credential);
            console.log(data);
        }
      </script>
  </body>
</html>
```
回傳的 data 內容物
```json
{
// data
  "iss": "https://accounts.google.com", // The JWT's issuer
  "nbf":  161803398874,
  "aud": "314159265-pi.apps.googleusercontent.com", // Your server's client ID
  "sub": "3141592653589793238", // The unique ID of the user's Google Account
  "hd": "gmail.com", // If present, the host domain of the user's GSuite email address
  "email": "elisa.g.beckett@gmail.com", // The user's email address
  "email_verified": true, // true, if Google has verified the email address
  "azp": "314159265-pi.apps.googleusercontent.com",
  "name": "Elisa Beckett",
                            // If present, a URL to user's profile picture
  "picture": "https://lh3.googleusercontent.com/a-/e2718281828459045235360uler",
  "given_name": "Eliza",
  "family_name": "Beckett",
  "iat": 1596474000, // Unix timestamp of the assertion's creation time
  "exp": 1596477600, // Unix timestamp of the assertion's expiration time
  "jti": "abc161803398874def"
}
```

#### Reference
https://vii120.coderbridge.io/2022/06/23/google-signin-gsi/
https://developers.google.com/identity/gsi/web/guides/display-button?hl=zh-cn