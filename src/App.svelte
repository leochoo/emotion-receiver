<script>
  import logo from "./assets/svelte.png";
  import Counter from "./lib/Counter.svelte";
  import { db } from "../firebaseConfig.js";

  import { doc, onSnapshot } from "firebase/firestore";

  // const pcaPath = require.resolve("../node_modules/@chirimen/pca9685/index.js");
  // console.log("pcaPath:", pcaPath);
  // const poly = require("../node_modules/@chirimen-raspi/polyfill/polyfill.js");

  // import { PCA9685 } from "../node_modules/@chirimen/pca9685/pca9685.js";
  // import { requestI2CAccess } from "../node_modules/@chirimen-raspi/polyfill/polyfill.js";

  // import { PCA9685 } from "@chirimen/pca9685/pca9685";

  // import * as pca from "@chirimen/pca9685";
  // import * as poly from "@chirimen-raspi/polyfill";

  let score = 0;

  let angleState = 0;

  const unsub = onSnapshot(doc(db, "events", "testEvent"), (doc) => {
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

<svelte:head>
  <script src="https://cdn.jsdelivr.net/npm/@chirimen-raspi/polyfill"></script>
  <script src="https://cdn.jsdelivr.net/npm/@chirimen/pca9685"></script>
</svelte:head>

<main>
  <img src={logo} alt="Svelte Logo" />
  <h1>Emotion Receiver</h1>

  <p>みんなの応援が届いてるよ！</p>

  <p>Feel your audience cheering for you!</p>

  <h2>Score {score}</h2>
  {#if score > 10000}
    <h1>FLY!!!</h1>
  {/if}
  <div>Angle Text: {angleState}</div>
  <button on:click={trigger}>Fly me</button>
</main>

<style>
  :root {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen,
      Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif;
  }

  main {
    text-align: center;
    padding: 1em;
    margin: 0 auto;
  }

  img {
    height: 16rem;
    width: 16rem;
  }

  h1 {
    color: #ff3e00;
    text-transform: uppercase;
    font-size: 4rem;
    font-weight: 100;
    line-height: 1.1;
    margin: 2rem auto;
    max-width: 14rem;
  }

  p {
    max-width: 14rem;
    margin: 1rem auto;
    line-height: 1.35;
  }

  @media (min-width: 480px) {
    h1 {
      max-width: none;
    }

    p {
      max-width: none;
    }
  }
</style>
