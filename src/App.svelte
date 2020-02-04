<script>
  import {onMount} from "svelte";
  import * as cocossd from "@tensorflow-models/coco-ssd";

  let classification;
  let boot;
  let model;
  let stream;
  let video;

  onMount(() => {
    boot = Promise.all([
      cocossd.load().then((_model) => (model = _model)),
      navigator.mediaDevices.getUserMedia({video: true}).then((_stream) => (stream = _stream)),
    ]);
  });

  function detect(e) {
    classification = model.detect(video).then((detections) => detections.map((detection) => detection.class));
  }

  $: stream, video && (video.srcObject = stream);
</script>

<style>
  main {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }
</style>

<main>
  {#await boot}
    <span>Loading model...</span>
  {:then}
    {#await classification}
      <span>Detecting...</span>
    {:then result}
      <video autoplay bind:this={video} />
      <button on:click={detect}>Detect</button>
      {#if result}
        <div>{JSON.stringify(result)}</div>
      {:else}
        <span>Click on Detect</span>
      {/if}
    {/await}
  {/await}
</main>
