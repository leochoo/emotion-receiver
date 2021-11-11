<script>
  export let chat_id;
  import BoardImage from "./imgs/CMMB_Embed.png";
  let name;
  var microBitBle;

  var gpioAccess;
  var mbGpioPorts;

  let sensorInfo;
  let gx;
  let gy;
  let gz;

  // LED
  var gpio0Val;
  var gpio1Val;
  var gpio2Val;
  var gpioPort0;
  var gpioPort1;
  var gpioPort2;

  async function connect() {
    microBitBle = await microBitBleFactory.connect();
    msg.innerHTML = "micro:bit BLE接続しました。";

    gpioAccess = await microBitBle.requestGPIOAccess();
    console.log("gpio", gpioAccess.ports);
    mbGpioPorts = gpioAccess.ports;

    gpioPort0 = mbGpioPorts.get(0);
    gpioPort1 = mbGpioPorts.get(1);
    gpioPort2 = mbGpioPorts.get(2);
  }

  async function disconnect() {
    await microBitBle.disconnect();
    msg.innerHTML = "micro:bit BLE接続を切断しました。";
  }

  let reading = false;

  const waitFor = (delay) =>
    new Promise((resolve) => setTimeout(resolve, delay));

  let prevAcc = null;
  let currAcc = null;
  let diffX = 0;
  let diffY = 0;
  let diffZ = 0;
  let maxDirection = "";
  let maxDiff = 0;

  let totalScore = 0;

  async function readSensor() {
    // toggle reading state
    reading = !reading;
    console.log("readSensor", reading);
    // if reading state true
    if (reading) {
      // for every second, fetch the sensor data!
      for (let i = 0; i < 10; i++) {
        var sdat = await microBitBle.readSensor();

        gx = sdat.acceleration.x;
        gy = sdat.acceleration.y;
        gz = sdat.acceleration.z;

        if (i === 0 && prevAcc === null) {
          prevAcc = sdat.acceleration;
          console.log("prevAcc", prevAcc);
        }
        if (i === 1 && currAcc === null) {
          currAcc = sdat.acceleration;
          console.log("currAcc", currAcc);
        }

        if (prevAcc !== null && currAcc !== null) {
          diffX = Math.abs(currAcc.x - prevAcc.x);
          diffY = Math.abs(currAcc.y - prevAcc.y);
          diffZ = Math.abs(currAcc.z - prevAcc.z);

          // update currAcc and prevAcc to the next sensor data
          prevAcc = currAcc;
          currAcc = sdat.acceleration;

          // take the maximum of diffX, diffY, diffZ
          if (diffX > diffY && diffX > diffZ) {
            maxDirection = "x";
            maxDiff = diffX;

            gpio0Val = 1;
            gpio1Val = 0;
            gpio2Val = 0;
          } else if (diffY > diffX && diffY > diffZ) {
            maxDirection = "y";
            maxDiff = diffY;

            gpio0Val = 0;
            gpio1Val = 1;
            gpio2Val = 0;
          } else {
            maxDiff = diffZ;
            maxDirection = "z";
            gpio0Val = 0;
            gpio1Val = 0;
            gpio2Val = 1;
          }
          console.log(maxDirection, maxDiff);

          await gpioPort0.write(gpio0Val);
          await gpioPort1.write(gpio1Val);
          await gpioPort2.write(gpio2Val);

          // update totalScore
          totalScore += maxDiff;
          await handleScore();
        }

        // console.log("i", i);
        // console.log("sensor:", sdat);
        // console.log("console.log:", sdat.acceleration.x);
        // console.log("diffX", diffX);

        if (reading == false) {
          break;
        }
        await waitFor(1000);
      }
    }
  }

  async function print() {
    await microBitBle.printLED(txt.value);
  }

  async function showIcon() {
    await microBitBle.showIconLED(Number(txt2.value));
  }

  import { doc, setDoc, getDoc, updateDoc } from "firebase/firestore";
  import { db } from "../firebase.js";
  import { onMount } from "svelte";

  onMount(async () => {
    // fetch initial totalScore from Firebase onMount
    const docRef = doc(db, "chats", chat_id);
    const docSnap = await getDoc(docRef);

    if (docSnap.exists()) {
      console.log("Document data:", docSnap.data());
      totalScore = docSnap.data().totalScore;
    } else {
      // doc.data() will be undefined in this case
      console.log("No such document!");
    }
  });

  async function handleScore() {
    // console.log("sending score to firebase");
    // await setDoc(doc(db, "events", "testEvent"), {
    //   totalScore: totalScore,
    // });
    console.log("sending score to firebase");
    await updateDoc(doc(db, "chats", chat_id), {
      totalScore: totalScore,
    });
  }
</script>

<main>
  <form name="js">
    <button type="button" class="btn btn-primary" on:click={connect}
      >Connect</button
    >
    <button type="button" class="btn btn-danger" on:click={disconnect}
      >Disconnect</button
    >
  </form>

  {#if totalScore}
    <h2>Total Score: {totalScore}</h2>
  {/if}
  <div id="msg">---</div>
  <hr />
  <div>
    <table>
      <tr>
        <td colspan="2">
          {#if reading == false}
            <button type="button" class="btn btn-info" on:click={readSensor}
              >Read Sensors (10s)</button
            >
          {:else}
            <button type="button" class="btn btn-danger" on:click={readSensor}
              >Stop Reading</button
            >
          {/if}
        </td>
      </tr>

      <tr>
        <td>Sensor Info Table</td>
      </tr>
      <tr>
        <td>Gx</td>
        <td>{gx}</td>
      </tr>
      <tr>
        <td>Gy</td>
        <td>{gy}</td>
      </tr>
      <tr>
        <td>Gz</td>
        <td>{gz}</td>
      </tr>
      <tr>
        <td colspan="2">LED</td>
      </tr>
      <tr>
        <td><input id="txt" type="text" value="Hello!" /></td>
        <td
          ><button type="button" color="secondary" on:click={print}
            >Print</button
          ></td
        >
      </tr>
      <tr>
        <td>
          <input id="txt2" style="width: 50px" type="text" value="0" />(0..39)
        </td>
        <!-- <td><input type="button" value="showIcon" on:click={showIcon} /></td> -->
        <td
          ><button type="button" color="secondary" on:click={showIcon}
            >Show Icon</button
          ></td
        >
      </tr>
    </table>
  </div>
  <img src={BoardImage} alt="random board" width="300" />
</main>

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
