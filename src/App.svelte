<script lang="ts">
  import Nav from "./lib/nav.svelte";
  import Login from "./lib/Login.svelte";
  import { currentUser, pb } from "./lib/pokectbase";
  import Messages from "./lib/Messages.svelte";
  import Tickets from "./lib/Tickets.svelte";
  function signOut() {
    pb.authStore.clear();
    //console.log("logout");
  }

  function switchTab(tab) {
    switch (tab) {
      case "Chat":
        break;

      case "Tickets":
        break;

      case "Chat":
        break;

      default:
        break;
    }
  }
</script>

<div class=" bg-gray-700 absolute h-full w-screen">
  {#if $currentUser}
    <div class="flex">
      <div
        class="fixed top-0 left-0 h-screen w-16 m-0
            flex flex-col
            bg-gray-900 text-white shadow-lg"
      >
        <button class="nav-icon" on:click={() => switchTab("Chat")}>
          <Nav
            width="20"
            height="20"
            icon="material-symbols:chat"
            tooltip="Chat"
            clazz="nav-icon"
          />
        </button>
        <button class="nav-icon">
          <Nav
            width="32"
            height="32"
            icon="bi:plus"
            tooltip="tooltip"
            clazz="nav-icon"
          />
        </button>
        <button class="nav-icon">
          <Nav
            width="20"
            height="20"
            icon="bi:lightning-fill"
            tooltip="tooltip"
            clazz="nav-icon"
          />
        </button>
        <button class="nav-icon">
          <Nav
            width="20"
            height="20"
            icon="fa6-solid:poo"
            tooltip="tooltip"
            clazz="nav-icon"
          />
        </button>
        <button class="nav-icon-logout" on:click={signOut}>
          <Nav
            width="20"
            height="20"
            icon="fa6-solid:x"
            tooltip="Logout"
            clazz="nav-icon-logout"
          />
        </button>
      </div>
    </div>
    <Tickets />
    <!--<Messages />-->
  {:else}
    <div class="flex items-center justify-center h-screen bg-gray-700">
      <div class="bg-gray-900 text-white shadow-lg w-72 h-72 login-box">
        <h3 class="flex items-center justify-center m-6 mb-0 text-green-500">
          FRCS Chat
        </h3>
        <Login />
      </div>
    </div>
  {/if}
</div>

<style lang="postcss">
</style>
