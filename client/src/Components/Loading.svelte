<script lang="ts">
  export let color: string = "black"
  export let heading: string = "Loading Data..."
</script>

<main>
	<div class="loading-container" style={`color: ${color};`}>
    <div>
      <div>{heading}</div>
  
      <div class="spinner">
        <div class="rect1" style={`background-color: ${color};`}></div>
        <div class="rect2" style={`background-color: ${color};`}></div>
        <div class="rect3" style={`background-color: ${color};`}></div>
        <div class="rect4" style={`background-color: ${color};`}></div>
        <div class="rect5" style={`background-color: ${color};`}></div>
      </div>
    </div>
  </div>
</main>

<style>
  .loading-container {
    display: flex;
    justify-content: center;
    align-items: center;
    font-weight: bold;
  }

  .spinner {
    margin: 20px auto;
    width: 50px;
    height: 40px;
    text-align: center;
    font-size: 10px;
  }

  .spinner > div {
    height: 100%;
    width: 4px;
    display: inline-block;
    margin-right: 2px;

    -webkit-animation: sk-stretchdelay 1.2s infinite ease-in-out;
    animation: sk-stretchdelay 1.2s infinite ease-in-out;
  }

  .spinner .rect2 {
    -webkit-animation-delay: -1.1s;
    animation-delay: -1.1s;
  }

  .spinner .rect3 {
    -webkit-animation-delay: -1.0s;
    animation-delay: -1.0s;
  }

  .spinner .rect4 {
    -webkit-animation-delay: -0.9s;
    animation-delay: -0.9s;
  }

  .spinner .rect5 {
    -webkit-animation-delay: -0.8s;
    animation-delay: -0.8s;
  }

  @-webkit-keyframes sk-stretchdelay {
    0%, 40%, 100% { -webkit-transform: scaleY(0.4) }
    20% { -webkit-transform: scaleY(1.0) }
  }

  @keyframes sk-stretchdelay {
    0%, 40%, 100% {
      transform: scaleY(0.4);
      -webkit-transform: scaleY(0.4);
    }  20% {
      transform: scaleY(1.0);
      -webkit-transform: scaleY(1.0);
    }
  }
</style>