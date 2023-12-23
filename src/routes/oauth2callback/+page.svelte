<script lang="ts">
  import { onMount } from "svelte";
  import { page } from "$app/stores";

  // 從 URL 字串中解析出訪問令牌
  function getAccessTokenFromString(str: string) {
    const regex = /#access_token=([^&]*)/;
    const match = str.match(regex);
    return match ? match[1] : null;
  }

  // 獲取當前頁面 URL 的訪問令牌
  const accessToken = getAccessTokenFromString($page.url.hash);

  // 這裡放置你的 API 金鑰
  const key = 'key'

  // 存放訂閱頻道的陣列
  let subscriptions: any[] = [];

  // 頁面加載時執行
  onMount(() => {
    // 從 YouTube API 獲取訂閱頻道
    fetch(
      `https://youtube.googleapis.com/youtube/v3/subscriptions?part=snippet&mine=true&key=${key}&maxResults=50`,
      {
        method: "GET",
        headers: {
          Authorization: "Bearer " + accessToken,
        },
      },
      )
      .then((response) => {
        console.log(response);
        return response.json();
      })
      .then((data) => {
        subscriptions.push(...data.items);
        subscriptions = [...subscriptions];
        return nextPage(data.nextPageToken);
      })
      .catch((error) => {
        console.log(error);
      });
  })
      
  // 處理分頁，獲取更多訂閱頻道
  function nextPage(token: string) {
    if (token) {
      console.log(token);
      fetch(
        `https://youtube.googleapis.com/youtube/v3/subscriptions?part=snippet&mine=true&key=${key}c&maxResults=50&pageToken=${token}`,
        {
          method: "GET",
          headers: {
            Authorization: "Bearer " + accessToken,
          },
        },
      )
        .then((response) => {
          console.log(response);
          return response.json();
        })
        .then((data) => {
          subscriptions.push(...data.items);
          subscriptions = [...subscriptions];
          return nextPage(data.nextPageToken);
        })
    }
  }

  // 取消訂閱指定的頻道
  function unsubscribe(id: string) {
    fetch(
        `https://youtube.googleapis.com/youtube/v3/subscriptions?id=${id}`,
        {
          method: "DELETE",
          headers: {
            Authorization: "Bearer " + accessToken,
          },
        },
      )
        .then((response) => {
          console.log(response);
          if(response.status == 204) {
            subscriptions = subscriptions.filter((sub) => sub.id != id)
          }
        })
  }

  // 取消訂閱所有頻道
  function unsubscribeAll() {
    if (window.confirm("Do you really want to unsubscribe all channels?")) {
      let i = 0
      subscriptions.forEach((sub) => {
        setTimeout(() => {
          unsubscribe(sub.id)
          console.log(`unsubscribe ${sub.snippet.title}`);
        }, 2000 * i);
        i++
      })
    }
  }
</script>

  <h1 class="text-center text-4xl mt-4">You have {subscriptions.length} subscriptions</h1>
  
  <ul>
    
    {#each subscriptions as sub}
    <li class="m-12 border-2 p-4 flex flex-col">
      <a href="https://www.youtube.com/channel/{sub.snippet.resourceId.channelId}" target="_blank" class="self-center">
        <img src="{sub.snippet.thumbnails.default.url}" alt="" class="h-24"> 
      </a>
      <h2 class="text-center text-xl p-2">{sub.snippet.title}</h2>
      <p>{sub.snippet.description}</p>
      <button class="btn btn-warning mt-2 self-end" on:click={()=>{unsubscribe(sub.id)}}>unsubscribe</button>
    </li>
    {/each}

    <button class="btn btn-error w-full" on:click={unsubscribeAll}>Unsubscribe All</button>
  </ul>
