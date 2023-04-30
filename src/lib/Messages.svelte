<script lang="ts">
  import { onMount, onDestroy } from "svelte";
  import { currentUser, pb } from "./pokectbase";
  import Icon from "@iconify/svelte";

  let newMessage: string;
  let messages = [];
  let unsubscribe: () => void;
  let index: number = 1;
  let isDisabledb: boolean = true;
  let isDisabledf: boolean = false;
  let innerHeight;
  let numMessages;

  $: numMessages = Math.floor((innerHeight - 72) / 57);

  $: {
    if (messages.length >= numMessages) {
      isDisabledf = false;
      console.log("dyn ", isDisabledf);
    }
  }

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
          console.log(messages.length);
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
    let totalPages = Math.ceil(messages.length / numMessages);
    // console.log(messages.length);
    //console.log(mid);

    if (!(index >= totalPages)) {
      //console.log('index+')
      index++;
      isDisabledb = false;
      if (index >= totalPages) {
        isDisabledf = true;
        console.log(isDisabledb);
        console.log(isDisabledf);
      }
    } else {
      isDisabledf = true;
      console.log(isDisabledb);
      console.log(isDisabledf);
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
        console.log(isDisabledb);
        console.log(isDisabledf);
      }
    } else {
      //console.log('back is disabled');
      isDisabledb = true;
      console.log(isDisabledb);
      console.log(isDisabledf);
    }
  }

  //console.log(isDisabledb);
  //console.log(isDisabledf);
</script>

<svelte:window bind:innerHeight />
<div class="msg-component">
  <div class="messages">
    {#each messages.slice().reverse() as message, id (message.id)}
      {#if id < index * numMessages && id >= index * numMessages - numMessages}
        <div class="msg">
          <img
            class="avatar"
            src="https://api.dicebear.com/6.x/personas/svg?seed={message.expand
              ?.user?.username}"
            alt="avatar"
            width="40px"
          />
          <div>
            <small class="select-none">
              Sent by @{message.expand?.user?.username}
            </small>
            <p class="msg-text">{message.text}</p>
          </div>
          {#if $currentUser.username == message.expand?.user?.username}
            <button
              on:click={() => deleteMessage(message.id)}
              on:keypress={() => deleteMessage(message.id)}
              class="delete-msg"
            >
              <Icon
                icon="fa6-solid:x"
                width="10"
                height="16"
                class="w-5 h-5 scale-50"
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
      <button class="login-button select-none" type="submit">Send</button>
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
