<script>
  export let id;
  import { Router, Link, Route } from "svelte-routing";
  import Sensor from "./Sensor.svelte";
  import { doc, onSnapshot } from "firebase/firestore";
  import { db } from "../firebase.js";
  import { onMount } from "svelte";

  let score;
  const unsub = onSnapshot(doc(db, "chats", id), (doc) => {
    console.log("Current data: ", doc.data());
    score = doc.data().totalScore;
  });
</script>

<main>
  <Link to="/">Back</Link>
  <h3>Welcome to ChatRoom {id}</h3>
  <Sensor chat_id={id} />
</main>
