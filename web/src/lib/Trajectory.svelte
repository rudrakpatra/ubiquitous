<script lang="ts">
  import { onMount, onDestroy } from "svelte";
  import { spring } from "svelte/motion";

  let isRecording = false;

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
    // points += 1;
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

  type Vector = { x: number; y: number; z: number };
  type Path = Array<Vector>;
  let displacement: Path = [{ x: 0, y: 0, z: 0 }];
  let velocity: Path = [{ x: 0, y: 0, z: 0 }];
  let acceleration: Path = [{ x: 0, y: 0, z: 0 }];

  const addScaledVector = (a: Vector, b: Vector, s: number) => {
    return { x: a.x + b.x * s, y: a.y + b.y * s, z: a.z + b.z * s };
  };

  const integrateMotion = (motion: DeviceMotionEvent) => {
    const acc = motion?.acceleration;
    const delta = motion?.interval;
    if (acc && delta) {
      points -= 1;
      acceleration.push({
        x: acc.x ? acc.x : 0,
        y: acc.y ? acc.y : 0,
        z: acc.z ? acc.z : 0,
      });
      acceleration =
        acceleration.length > 100 ? acceleration.splice(1) : acceleration;
      velocity.push(
        addScaledVector(
          velocity[velocity.length - 1],
          acceleration[acceleration.length - 1],
          delta
        )
      );
      velocity = velocity.length > 100 ? velocity.splice(1) : velocity;
      displacement.push(
        addScaledVector(
          displacement[displacement.length - 1],
          velocity[velocity.length - 1],
          delta
        )
      );
      displacement =
        displacement.length > 100 ? displacement.splice(1) : displacement;
    }
  };
  $: integrateMotion(motion);
</script>

<h3>Trajectory Demo</h3>
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
  <button
    on:click={() => {
      velocity = [{ x: 0, y: 0, z: 0 }];
      displacement = [{ x: 0, y: 0, z: 0 }];
      acceleration = [{ x: 0, y: 0, z: 0 }];
    }}
  >
    reset
  </button>
  <!-- <p>
    {displacement[displacement.length - 1].x.toFixed(2)},
    {displacement[displacement.length - 1].y.toFixed(2)},
    {displacement[displacement.length - 1].z.toFixed(2)}
  </p>
  <p>
    {velocity[velocity.length - 1].x.toFixed(2)},
    {velocity[velocity.length - 1].y.toFixed(2)},
    {velocity[velocity.length - 1].z.toFixed(2)}
  </p> -->
  <p>
    {acceleration[acceleration.length - 1].x.toFixed(2)},
    {acceleration[acceleration.length - 1].y.toFixed(2)},
    {acceleration[acceleration.length - 1].z.toFixed(2)}
  </p>
  <svg viewBox="-100 -100 200 200">
    <polyline
      points={displacement.map((pt) => `${pt.x * 0.1},${pt.y * 0.1}`).join(" ")}
      stroke="#0002"
      fill="none"
    />
    <polyline
      points={velocity.map((pt) => `${pt.x * 0.4},${pt.y * 0.4}`).join(" ")}
      stroke="#00f8"
      fill="none"
    />
    <polyline
      points={acceleration.map((pt) => `${pt.x * 10},${pt.y * 10}`).join(" ")}
      stroke="#f008"
      fill="none"
    />
    <circle
      cx={displacement[displacement.length - 1].x * 0.1}
      cy={displacement[displacement.length - 1].y * 0.1}
      r="2"
      fill="#000"
    />
    <circle
      cx={velocity[velocity.length - 1].x * 0.4}
      cy={velocity[velocity.length - 1].y * 0.4}
      r="2"
      fill="#00f"
    />
    <circle
      cx={acceleration[acceleration.length - 1].x * 10}
      cy={acceleration[acceleration.length - 1].y * 10}
      r="2"
      fill="#f00"
    />
  </svg>
{/if}
<p style="color:#f008">
  acc <strong>10x</strong>
</p>
<p style="color:#00f8">
  vel <strong>0.4x</strong>
</p>
<p style="color:#0002">
  pos <strong>0.1x</strong>
</p>

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

  svg {
    background-color: #f4f4f4;
    border-radius: 5px;
    width: 100%;
    height: 100%;
  }
</style>