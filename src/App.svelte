<script>
  import "bootswatch/dist/litera/bootstrap.min.css";
  import Home from "./Home.svelte";
  import Login from "./Login.svelte";
  import ChatList from "./ChatList.svelte";
  import ChatRoom from "./ChatRoom.svelte";

  import { Router, Link, Route } from "svelte-routing";

  import { userStatus } from "./stores";

  let user_status;
  userStatus.subscribe((value) => {
    user_status = value;
    console.log("user status changed", value);
  });
</script>

<svelte:head>
  <script src="https://cdn.jsdelivr.net/npm/@chirimen-raspi/polyfill"></script>
  <script src="https://cdn.jsdelivr.net/npm/@chirimen/pca9685"></script>
</svelte:head>

<Router>
  <main>
    <Home />
    <Login />
    <!-- <div>user_status: {user_status}</div> -->

    <Route path="chat/:id" let:params>
      <ChatRoom id={params.id} />
    </Route>

    <Route path="/">
      {#if user_status}
        <div>
          Welcome {user_status["uid"]} !
        </div>
        <ChatList uid={user_status.uid} />
      {/if}
    </Route>
  </main>
</Router>

<style>
  main {
    text-align: center;
    padding: 1em;
    margin: 0 auto;
  }
  h1 {
    color: #ff3e00;
    text-transform: uppercase;
    font-size: 4em;
    font-weight: 100;
  }
  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }

  :root {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen,
      Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif;
  }
</style>
