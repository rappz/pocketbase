<script lang="ts">
  import { currentUser, pb } from "./pokectbase";

  let username: string;
  let password: string;
  let name: string;
  let email: string;
  let newSignUp: boolean = false;

  async function login() {
    await pb.collection("users").authWithPassword(username, password);
    console.log($currentUser.username);
  }

  async function signUpSubmit() {
    try {
      const data = {
        username,
        password,
        passwordConfirm: password,
        name,
        email,
      };
      const createdUser = await pb.collection("users").create(data);
      await login();
      newSignUp = false;
    } catch (err) {
      console.error(err);
    }
  }
  export function signOut() {
    pb.authStore.clear();
  }
</script>

<div class="flex items-center justify-center h-full w-full mt-0">
  {#if !$currentUser}
    <form on:submit|preventDefault>
      <input
        placeholder="Username"
        type="text"
        bind:value={username}
        class="login-text flex justify-center"
        required
      />

      <input
        placeholder="Password"
        type="password"
        bind:value={password}
        class="login-text flex justify-center"
        required
      />
      {#if newSignUp}
        <!-- content here -->
        <input
          placeholder="Name"
          type="text"
          bind:value={name}
          class="login-text flex justify-center"
          required
        />

        <input
          placeholder="Email"
          type="email"
          bind:value={email}
          class="login-text flex justify-center"
          required
        />
      {/if}
      <div class="flex">
        {#if newSignUp}
          <button class="login-button" on:click={signUpSubmit}>Submit</button>
        {:else}
          <button class="login-button" on:click={login}>Login</button>
          <button class="login-button" on:click={() => (newSignUp = true)}
            >Sign Up</button
          >
        {/if}
      </div>
    </form>
  {/if}
</div>
