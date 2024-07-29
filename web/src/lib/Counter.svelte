<script lang="ts">
  import { onMount, onDestroy } from "svelte";

  let isRecording = false;
  const requestPermissionForIOS = () => {
    // Request permission for iOS 13+ devices
    if (
      DeviceMotionEvent &&
      // @ts-ignore
      typeof DeviceMotionEvent.requestPermission === "function"
    ) {
      // @ts-ignore
      DeviceMotionEvent.requestPermission();
    } else {
      alert("DeviceMotionEvent not supported");
    }
    if (
      DeviceOrientationEvent &&
      // @ts-ignore
      typeof DeviceOrientationEvent.requestPermission === "function"
    ) {
      // @ts-ignore
      DeviceOrientationEvent.requestPermission();
    } else {
      alert("DeviceOrientationEvent not supported");
    }
  };

  let motion: DeviceMotionEvent;
  let orientation: DeviceOrientationEvent;

  let alpha: number;
  let beta: number;
  let gamma: number;

  let accX: number;
  let accY: number;
  let accZ: number;

  let interval: number;
  let points = 0;
  $: if (!isRecording) points = 0;

  $: alpha = orientation?.alpha ? orientation.alpha : alpha;
  $: beta = orientation?.beta ? orientation.beta : beta;
  $: gamma = orientation?.gamma ? orientation.gamma : gamma;

  $: accX = motion?.acceleration?.x ? motion.acceleration.x : accX;
  $: accY = motion?.acceleration?.y ? motion.acceleration.y : accY;
  $: accZ = motion?.acceleration?.z ? motion.acceleration.z : accZ;

  $: interval = motion ? motion.interval : interval;

  const handleMotion = (event: DeviceMotionEvent) => {
    motion = event;
    points += 1;
  };

  const handleOrientation = (event: DeviceOrientationEvent) => {
    orientation = event;
    points += 1;
  };

  const toggleRecording = () => {
    isRecording = !isRecording;
    ondevicemotion = isRecording ? handleMotion : null;
    ondeviceorientationabsolute = isRecording ? handleOrientation : null;
  };

  onMount(() => {
    // requestPermissionForIOS();
    //lock orientation
    // @ts-ignore
    // screen.orientation.lock("portrait");
  });

  onDestroy(() => {
    ondevicemotion = null;
    ondeviceorientation = null;
  });
</script>

<h3>Device Motion and Orientation Demo</h3>
<h6>
  <strong>Tip:</strong> Open this page on a mobile device to see the motion data.
</h6>

<button on:click|preventDefault={toggleRecording}>
  {isRecording ? "Stop Demo" : "Start Demo"}
</button>
{#if isRecording}
  <h4>points</h4>
  <p>
    {points}
  </p>
  <h3>orientation</h3>
  <h4>alpha (around Z)</h4>
  <p>
    0
    <input type="range" min="0" max="360" value={alpha} step="0.1" disabled />
    360
  </p>
  <h4>beta (around X)</h4>
  <p>
    -180
    <input type="range" min="-180" max="180" value={beta} step="0.1" disabled />
    180
  </p>
  <h4>gamma (around Y)</h4>
  <p>
    -90
    <input type="range" min="-90" max="90" value={gamma} step="0.1" disabled />
    90
  </p>
  <h3>acceleration</h3>
  <h4>accX</h4>
  <p>
    -10
    <input type="range" min="-10" max="10" value={accX} step="0.1" disabled />
    10
  </p>
  <h4>accY</h4>
  <p>
    -10
    <input type="range" min="-10" max="10" value={accY} step="0.1" disabled />
    10
  </p>
  <h4>accZ</h4>
  <p>
    -10
    <input type="range" min="-10" max="10" value={accZ} step="0.1" disabled />
    10
  </p>
  <h3>interval</h3>
  <p>{interval}</p>
{/if}

<style>
  button {
    background-color: #e2f2a2;
    border: none;
    border-radius: 5px;

    margin: 10px 0;
    padding: 10px 20px;
    font-size: 16px;
    cursor: pointer;
  }

  p {
    display: flex;
    align-items: center;
    background-color: #f4f4f4;
    padding: 10px;
    border-radius: 5px;
    overflow-x: auto;
  }
</style>
