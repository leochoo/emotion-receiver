<script>
  export let id;
  import { Router, Link, Route } from "svelte-routing";
  import Sensor from "./Sensor.svelte";
  import { doc, onSnapshot } from "firebase/firestore";
  import { db } from "../firebase.js";
  import { onMount } from "svelte";

  let score;
  let angleState = 0;

  const unsub = onSnapshot(doc(db, "chats", id), (doc) => {
    console.log("Current data: ", doc.data());
    score = doc.data().totalScore;
  });

  async function trigger() {
    var i2cAccess = await navigator.requestI2CAccess();
    var port = i2cAccess.ports.get(1);
    var pca9685 = new PCA9685(port, 0x40);

    console.log("i2cAccess", i2cAccess);
    console.log("PCA9685: ", pca9685);

    var angle = 0;
    // console.log("angle"+angle);
    // servo setting for sg90
    // Servo PWM pulse: min=0.0011[sec], max=0.0019[sec] angle=+-60[deg]
    await pca9685.init(0.001, 0.002, 30);
    for (let i = 0; i < 10; i++) {
      console.log("loop i", i);
      angle = angle <= -30 ? 30 : -30;
      // console.log("angle"+angle);
      await pca9685.setServo(0, angle);
      // console.log('value:', angle);
      angleState = angle;
      await sleep(1000);
    }
  }
</script>

<main>
  <Link to="/">Back</Link>
  <h3>Welcome to ChatRoom {id}</h3>
  <h2>Score {score}</h2>
  {#if score > 15000}
    <h1>FLY!!!</h1>
    {trigger()}
  {/if}

  <!-- <Sensor chat_id={id} /> -->
</main>
