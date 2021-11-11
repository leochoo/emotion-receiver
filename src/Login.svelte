<script>
  import { auth } from "../firebase.js";
  import {
    signInAnonymously,
    createUserWithEmailAndPassword,
    signInWithEmailAndPassword,
  } from "firebase/auth";
  import { authState } from "rxfire/auth";
  import { userStatus } from "./stores";

  let newUser = false;

  let user_status;
  userStatus.subscribe((value) => {
    user_status = value;
  });

  let user;
  const unsubscribe = authState(auth).subscribe((u) => {
    user = u;
    userStatus.set(user);
    // console.log("user_status subscribed", { user_status });
  });
  async function login() {
    // console.log("log in");
    await signInAnonymously(auth);
  }
  async function logout() {
    // console.log("log out");
    await auth.signOut();
  }

  let name;
  let email;
  let password;
  let errorMessage = "";
  async function signInOrCreateUser() {
    console.log(name, email, password);
    try {
      if (newUser) {
        await createUserWithEmailAndPassword(auth, email, password);
      } else {
        await signInWithEmailAndPassword(auth, email, password);
      }
    } catch (error) {
      errorMessage = error.message;
    }
  }
</script>

<main>
  {#if user_status}
    <button type="button" class="btn btn-dark" on:click={logout}>Logout</button>
  {:else}
    <button type="button" class="btn btn-dark" on:click={login}>
      Sign In Anonymously
    </button>

    {#if newUser}
      <div>
        <h3>Sign Up for a New Account</h3>
        <button class="btn btn-secondary" on:click={() => (newUser = false)}
          >Returning User?</button
        >
      </div>
    {:else}
      <div>
        <h3>Sign In with Email</h3>
        <button class="btn btn-secondary" on:click={() => (newUser = true)}
          >New User?</button
        >
      </div>
    {/if}
    <form>
      <fieldset>
        {#if newUser}
          <div class="form-group row">
            <label for="nameText" class="col-sm-2 col-form-label">Name</label>
            <div class="col-sm-10">
              <input
                type="text"
                class="form-control"
                placeholder="John Smith"
                bind:value={name}
              />
            </div>
          </div>
        {/if}
        <div class="form-group row">
          <label for="exampleInputEmail1" class="col-sm-2 col-form-label"
            >Email</label
          >
          <div class="col-sm-10">
            <input
              type="email"
              class="form-control"
              id="exampleInputEmail1"
              aria-describedby="emailHelp"
              placeholder="Enter email"
              bind:value={email}
            />
          </div>
        </div>

        <div class="form-group row">
          <label for="exampleInputPassword1" class="col-sm-2 col-form-label"
            >Password</label
          >
          <div class="col-sm-10">
            <input
              type="password"
              class="form-control"
              id="exampleInputPassword1"
              placeholder="Password"
              bind:value={password}
            />
          </div>
        </div>

        <button
          type="submit"
          class="btn btn-primary"
          on:click|preventDefault={signInOrCreateUser}>Submit</button
        >

        <p class="has-text-danger" v-if="errorMessage">{errorMessage}</p>
      </fieldset>
    </form>
  {/if}
  <hr />
</main>
