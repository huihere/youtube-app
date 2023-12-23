<script lang="ts">
  // 重定向 URI, 在 OAuth 驗證流程中使用
  const redirect_uri = 'http://localhost:5173/oauth2callback'
  const client_id = 'client_id'

  /*
   * 創建表單以從 Google 的 OAuth 2.0 伺服器請求訪問令牌。
   */
  function oauthSignIn() {
    // Google 的 OAuth 2.0 端點，用於請求訪問令牌
    var oauth2Endpoint = "https://accounts.google.com/o/oauth2/v2/auth";

    // 創建 <form> 元素，提交參數到 OAuth 2.0 端點。
    var form = document.createElement("form");
    form.setAttribute("method", "GET"); // 以 GET 請求發送。
    form.setAttribute("action", oauth2Endpoint);

    interface Params {
      [key: string]: string;
    }
    
    // 傳遞給 OAuth 2.0 端點的參數。
    var params: Params = {
      client_id,
      redirect_uri,
      response_type: "token",
      scope: "https://www.googleapis.com/auth/youtubepartner",
    };

    // 將表單參數添加為隱藏的輸入值。
    for (var p in params) {
      var input = document.createElement("input");
      input.setAttribute("type", "hidden");
      input.setAttribute("name", p);
      input.setAttribute("value", params[p]);
      form.appendChild(input);
    }

    // 將表單添加到頁面並提交，以打開 OAuth 2.0 端點。
    document.body.appendChild(form);
    form.submit();
  }
</script>

<h1 class="text-3xl m-4">Mass Unsubscribe YouToube</h1>
<p>
  <button class="btn btn-warning" on:click={oauthSignIn}>使用 Google 帳號登入</button>
</p>
