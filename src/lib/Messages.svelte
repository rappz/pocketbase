<script lang="ts">
  import { onMount, onDestroy } from "svelte";
  import { currentUser, pb } from "./pokectbase";
  import Icon from "@iconify/svelte";

  let selectedId: string;
  let newMessage: string;
  let messages = [];
  let unsubscribe: () => void;
  let index: number = 1;
  let isDisabledb: boolean = true;
  let isDisabledf: boolean = false;

  onMount(async () => {
    const resultList = await pb.collection("messages").getList(1, 250, {
      sort: "created",
      expand: "user",
    });
    messages = resultList.items;

    unsubscribe = await pb
      .collection("messages")
      .subscribe("*", async ({ action, record }) => {
        if (action === "create") {
          const user = await pb.collection("users").getOne(record.user);
          record.expand = { user };
          messages = [...messages, record];
        }
        if (action === "delete") {
          messages = messages.filter((m) => m.id !== record.id);
        }
      });
  });

  onDestroy(() => {
    unsubscribe();
  });
  async function sendMessage() {
    const data = {
      text: newMessage,
      user: $currentUser.id,
    };
    const createdMessage = await pb.collection("messages").create(data);
    newMessage = "";
  }
  async function deleteMessage(msg: string) {
    const deletedMessage = await pb
      .collection("messages")
      .delete(msg, $currentUser);
  }

  function nextPage() {
    let totalPages = Math.ceil(messages.length / 20);
    // console.log(messages.length);
    //console.log(mid);

    if (!(index >= totalPages)) {
      //console.log('index+')
      index++;
      isDisabledb = false;
      if (index >= totalPages) {
        isDisabledf = true;
      }
    } else {
      isDisabledf = true;
    }
  }
  function prevPage() {
    //console.log(index)

    if (!(index <= 1)) {
      index--;
      isDisabledf = false;
      if (index <= 1) {
        //console.log('back is disabled2');
        isDisabledb = true;
      }
    } else {
      //console.log('back is disabled');
      isDisabledb = true;
    }
  }

  console.log(isDisabledb);
  console.log(isDisabledf);
</script>

<div class="flex flex-col items-center justify-center h-screen">
  <div class="messages h-fit overflow-y-auto">
    {#each messages.slice().reverse() as message, id (message.id)}
      {#if id < index * 20 && id >= index * 20 - 20}
        {@const mid = id}
        <div class=" login-text msg flex flex-row">
          <img
            class="avatar"
            src="https://api.dicebear.com/6.x/personas/svg?seed={message.expand
              ?.user?.username}"
            alt="avatar"
            width="40px"
          />
          <div>
            <small>
              Sent by @{message.expand?.user?.username}
            </small>
            <p class="msg-text login-text">{message.text}</p>
          </div>
          {#if $currentUser.username == message.expand?.user?.username}
            <button
              on:click={() => deleteMessage(message.id)}
              on:keypress={() => deleteMessage(message.id)}
              class="inline-block h-fit w-fit justify-items-end ml-auto align-middle"
            >
              <Icon
                class="delete-msg justify-self-end align-middle"
                icon="fa6-solid:x"
                width="12"
                height="12"
              />
            </button>
          {/if}
        </div>
      {/if}
    {/each}
  </div>
  <span class="flex flex-row">
    {#key isDisabledb}
      <button
        disabled={isDisabledb}
        on:click={prevPage}
        class="nav-icon-no-size"
      >
        <Icon
          icon="material-symbols:arrow-back-ios-new-rounded"
          class="h-auto"
        />
      </button>
    {/key}
    <form class="flex" on:submit|preventDefault={sendMessage}>
      <input
        class="login-text flex items-center bg-gray-900"
        placeholder="Message"
        type="text"
        bind:value={newMessage}
      />
      <button class="login-button" type="submit">Send</button>
    </form>
    {#key isDisabledf}
      <button
        disabled={isDisabledf}
        on:click={nextPage}
        class="nav-icon-no-size"
      >
        <Icon
          icon="material-symbols:arrow-back-ios-new-rounded"
          rotate={2}
          class="h-auto"
        />
      </button>
    {/key}
  </span>
</div>
<!--

index 1 * 20= 20-20=0
start 0
end 20

index 2 * 20 = 40-20=20


make button have icon class instead of using invisible span2


-->
