<script>
  import Spinner from "./Spinner.svelte";

  const DAILY_BIBLE_FEED = "https://feeds.gty.org/gtydailybible&x=1";

  const domParser = new DOMParser();

  let title = new Intl.DateTimeFormat("en", {
    month: "long",
    day: "numeric",
  }).format(new Date());
  let copyright = "2017 by Grace to You";
  let readings = [];
  let loading = 0;
  let loaded = false;

  (async () => {
    const response = await fetch(DAILY_BIBLE_FEED);
    const feedText = await response.text();

    const feed = domParser.parseFromString(feedText, "text/xml");
    const feedItem = feed.querySelector("rss > channel > item:first-of-type");

    copyright = feed.querySelector("rss > channel > copyright").textContent;
    title = feedItem.querySelector("title").textContent;

    const contentText = feedItem.querySelector("description").textContent;
    const content = domParser.parseFromString(contentText, "text/html");

    readings = Array.from(content.querySelectorAll("ul:first-of-type li")).map(
      (elem) => elem.innerText
    );
    loading = readings.length;
  })();

  function handleAudioLoad() {
    loading--;
    if (loading == 0) {
      loaded = true;
    }
  }
</script>

<style>
  #root {
    min-height: 100%;
    display: flex;
    flex-direction: column;
    padding: 2rem;
    margin: 0 auto;
    max-width: 45rem;
  }

  header {
    text-align: center;
  }

  footer {
    margin: 2rem auto 0 auto;
    max-width: 28rem;
    text-align: center;
    line-height: 1.3;
  }

  a {
    color: inherit;
    opacity: 0.6;
  }

  h1 {
    margin: 0;
  }

  h2 {
    margin-top: 0;
    margin-bottom: 2rem;
    opacity: 0.6;
    font-weight: normal;
    font-size: 1.5rem;
  }

  main {
    flex-grow: 1;
  }

  .hide {
    display: none !important;
  }

  .audio-player {
    display: block;
    border: 0;
    width: 100%;
    height: 109px;
  }

  .audio-player:not(:first-child) {
    margin-top: 1rem;
  }

  .loading {
    flex-grow: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }

  @media screen and (min-height: 1000px) {
    h1 {
      margin-top: 2rem;
    }

    h2 {
      margin-bottom: 4rem;
    }

    .audio-player {
      margin-bottom: 2rem;
    }
  }
</style>

<div id="root">
  <header>
    <h1>{title}</h1>
    <h2>MacArthur Daily Bible</h2>
  </header>

  <main class:hide={!loaded}>
    {#each readings as reading}
      <iframe
        class="audio-player"
        title="Audio for {reading}"
        src="https://www.esv.org/audio-player/{reading}/"
        on:load={handleAudioLoad} />
    {/each}
  </main>

  {#if !loaded}
    <div class="loading">
      <Spinner />
    </div>
  {/if}

  <footer class:hide={!loaded}>
    Daily Bible readings from the one-year reading plan in the
    <a href="https://www.gty.org/library/devotionals/daily-bible">MacArthur
      Daily Bible</a>, copyright
    {copyright}. Built by
    <a href="https://github.com/ibelieve/gty-daily-audio-bible">Michael Spencer</a>.
  </footer>
</div>
