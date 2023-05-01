<script lang="ts">
  import { onMount, onDestroy } from "svelte";
  import { currentUser, pb } from "./pokectbase";
  import Icon from "@iconify/svelte";
  let innerHeight;
  let tickets = [];
  let unsubscribe: () => void;
  let selectedTicket={summary:"Summary", description:"Description",expand:{customer:{name:"Customer Name"},assignedTo:{name:"Technician Name"}},group:"Group Name",files: "None"};

  onMount(async () => {
    const resultList = await pb.collection("ticketing").getList(1, 250, {
      sort: "created",
      expand: "customer,assignedTo",
    });
    tickets = resultList.items;

    unsubscribe = await pb
      .collection("ticketing")
      .subscribe("*", async ({ action, record }) => {
        if (action === "create") {
          const user = await pb.collection("users").getOne(record.user);
          record.expand = { user };
          tickets = [...tickets, record];
          console.log(tickets.length);
        }
        if (action === "delete") {
          tickets = tickets.filter((m) => m.id !== record.id);
        }
      });
  });

  onDestroy(() => {
    unsubscribe();
  });

  function selectTicket(ticket){
    selectedTicket = ticket;
  }
</script>

<svelte:window bind:innerHeight />
<div class="ticket-component">
  <div class="ticketing">
    {#each tickets as ticket (ticket.id)}
      <button on:click={() => selectedTicket = ticket} class="ticket-button">
        <img
          class="avatar"
          src="https://api.dicebear.com/6.x/personas/svg?seed={ticket.expand
            ?.customer?.username}"
          alt="avatar"
          width="40px"
        />
        <div>
          <small class="relative ml-3 select-none">
            Sent by {ticket.expand?.customer?.username}
          </small>
          <p class="summary-text">{ticket.summary}</p>
        </div>
    </button>
    {/each}
  </div>

  {#key selectedTicket}
    
      <form on:submit|preventDefault on:submit|preventDefault class="ticketing-area">
        <input type="text" class=" ticket-text" placeholder="Summary" value={selectedTicket.summary} />
        <textarea class="ticket-text-area" placeholder="Description" value={selectedTicket.description} />
        <input class="nav-icon-no-size" type="file" placeholder="Attachments" />
        <p>Uploaded files: {selectedTicket.files}</p>
        <div class="flex items-center justify-center">
        <label for="assignedTo">Assigned To</label>
        <select class="assign-box" name="assignedTo">
            <option value={selectedTicket.expand?.assignedTo?.name}>{selectedTicket.expand?.assignedTo?.name}</option>
        </select>
        <label for="customer">Customer</label>
        <select  class="assign-box" name="customer">
            <option placeholder="Technician Name" value={selectedTicket.expand?.customer?.name}>{selectedTicket.expand?.customer?.name}</option>
        </select>
        <label for="group">Group</label>
        <select class="assign-box" name="group">
            <option value={selectedTicket.group}>{selectedTicket.group}</option>
        </select>
    </div>
        <button class=" bg-gray-700 w-20 flex justify-center ml-auto mr-3 mt-10" on:click={()=> console.log(selectedTicket)} type="submit">Update</button>
        <button class="bg-gray-700 w-20 flex justify-center ml-auto mr-3 mt-2" type="submit">Submit</button>
      </form>
    
  {/key}
</div>
