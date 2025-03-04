<script>
  import { onMount } from "svelte"
  import Translate from "$components/Translate.svelte"
  import { htmlToJsx } from "$lib/actions"
  let Prism
  onMount(async () => {
    Prism = (await import("./Prism.svelte")).default
  })
  export let title = undefined
  export let desc = undefined
  export let bg = undefined
  export let classes = undefined
  export let responsive = false
  let wrapper
  let showContent = "preview"
  let htmlSlot
  let jsxSlot

  let isClipboardButtonPressed = false
  const copyText = (text) => {
    navigator.clipboard.writeText(text)
    isClipboardButtonPressed = true
    setTimeout(() => {
      isClipboardButtonPressed = false
    }, 2000)
  }
  const decodeHtml = (html) => {
    var txt = document.createElement("textarea")
    txt.innerHTML = html
    return txt.value
  }
  $: titleStr = title
    ? title
        .replace(/[ ]/g, "-") // replace spaces with -
        .replace(/[^A-Za-z0-9-]/g, "") // replace all non-alphanumeric chars
        .toLowerCase()
    : ""

  onMount(() => {
    if (document.getElementById(location.hash.slice(1)) && location.hash.slice(1) == titleStr) {
      document.getElementById(location.hash.slice(1)).click()
    }
  })
</script>

<div
  class="component-preview not-prose text-base-content my-6 lg:my-12"
  id={titleStr}
  bind:this={wrapper}>
  {#if title}
    <div class="pb-2 text-sm font-bold">
      <a class="opacity-20 hover:opacity-60" href={`#${titleStr}`}>#</a>
      <span class="component-preview-title">{title}</span>
    </div>
  {/if}
  {#if desc}
    <div class="pb-2 text-xs opacity-70">
      {desc}
    </div>
  {/if}
  <div class="grid">
    <div class="tabs tabs-lifted z-10 -mb-[var(--tab-border)] justify-self-start">
      <button
        on:click={() => (showContent = "preview")}
        class={`tab ${
          showContent == "preview"
            ? "tab-active [--tab-bg:var(--fallback-b1,oklch(var(--b1)))]"
            : "[--tab-border-color:transparent]"
        }`}>
        <Translate text="Preview" />
      </button>
      <button
        on:click={() => (showContent = "html")}
        class={`tab ${
          showContent == "html"
            ? "tab-active [--tab-bg:var(--fallback-n,oklch(var(--n)))] [--tab-border-color:var(--fallback-n,oklch(var(--n)))] [--tab-color:var(--fallback-nc,oklch(var(--nc)))]"
            : "[--tab-border-color:transparent]"
        }`}>
        HTML
      </button>
      <button
        on:click={() => (showContent = "jsx")}
        class={`tab ${
          showContent == "jsx"
            ? "tab-active [--tab-bg:var(--fallback-n,oklch(var(--n)))] [--tab-border-color:var(--fallback-n,oklch(var(--n)))] [--tab-color:var(--fallback-nc,oklch(var(--nc)))]"
            : "[--tab-border-color:transparent]"
        }`}>
        JSX
      </button>
      <div class="tab [--tab-border-color:transparent]" />
    </div>

    {#if showContent == "preview"}
      <div class="bg-base-300 rounded-b-box rounded-se-box relative overflow-x-auto">
        <div
          class="preview border-base-300 bg-base-100 rounded-b-box rounded-se-box flex min-h-[6rem] min-w-[18rem] max-w-4xl flex-wrap items-center justify-center gap-2 overflow-x-hidden bg-cover bg-top p-4 [border-width:var(--tab-border)] {classes}"
          style={bg ? `background-image: url(${bg})` : ``}
          class:resize-x={responsive}>
          <slot />
        </div>
      </div>
    {/if}

    {#if onMount && showContent == "html"}
      <div class="grid">
        <div class="hidden" bind:this={htmlSlot}>
          <slot name="html" />
        </div>
        <div class="code-wrapper col-start-1 row-start-1">
          <svelte:component this={Prism} language="html">
            <slot name="html" />
          </svelte:component>
        </div>
        <div class="col-start-1 row-start-1 flex items-start justify-end p-2 rtl:justify-start">
          <div
            data-tip={isClipboardButtonPressed ? "copied" : "copy"}
            class="tooltip tooltip-left tooltip-accent">
            <button
              class="btn btn-square btn-sm btn-neutral"
              on:click={() => copyText(decodeHtml(htmlSlot.firstChild.innerHTML))}>
              {#if isClipboardButtonPressed}
                <svg
                  class="h-5 w-5 fill-current"
                  xmlns="http://www.w3.org/2000/svg"
                  viewBox="0 0 32 32">
                  <path
                    d="M 16 2 C 14.742188 2 13.847656 2.890625 13.40625 4 L 5 4 L 5 29 L 27 29 L 27 4 L 18.59375 4 C 18.152344 2.890625 17.257813 2 16 2 Z M 16 4 C 16.554688 4 17 4.445313 17 5 L 17 6 L 20 6 L 20 8 L 12 8 L 12 6 L 15 6 L 15 5 C 15 4.445313 15.445313 4 16 4 Z M 7 6 L 10 6 L 10 10 L 22 10 L 22 6 L 25 6 L 25 27 L 7 27 Z M 21.28125 13.28125 L 15 19.5625 L 11.71875 16.28125 L 10.28125 17.71875 L 14.28125 21.71875 L 15 22.40625 L 15.71875 21.71875 L 22.71875 14.71875 Z" />
                </svg>
              {:else}
                <svg
                  class="h-5 w-5 fill-current"
                  xmlns="http://www.w3.org/2000/svg"
                  viewBox="0 0 32 32">
                  <path
                    d="M 16 3 C 14.742188 3 13.847656 3.890625 13.40625 5 L 6 5 L 6 28 L 26 28 L 26 5 L 18.59375 5 C 18.152344 3.890625 17.257813 3 16 3 Z M 16 5 C 16.554688 5 17 5.445313 17 6 L 17 7 L 20 7 L 20 9 L 12 9 L 12 7 L 15 7 L 15 6 C 15 5.445313 15.445313 5 16 5 Z M 8 7 L 10 7 L 10 11 L 22 11 L 22 7 L 24 7 L 24 26 L 8 26 Z" />
                </svg>
              {/if}
            </button>
          </div>
        </div>
      </div>
    {/if}
    {#if onMount && showContent == "jsx"}
      <div class="grid">
        <div class="hidden" bind:this={jsxSlot}>
          {#if $$slots.jsx}
            <pre use:htmlToJsx><slot name="jsx" /></pre>
          {:else}
            <pre use:htmlToJsx><slot name="html" /></pre>
          {/if}
        </div>
        <div class="code-wrapper col-start-1 row-start-1">
          <svelte:component this={Prism} language="svelte">
            <div use:htmlToJsx>
              {#if $$slots.jsx}
                <slot name="jsx" />
              {:else}
                <slot name="html" />
              {/if}
            </div>
          </svelte:component>
        </div>
        <div class="col-start-1 row-start-1 flex items-start justify-end p-2">
          <div
            data-tip={isClipboardButtonPressed ? "copied" : "copy"}
            class="tooltip tooltip-left tooltip-accent">
            <button
              class="btn btn-square btn-sm btn-neutral"
              on:click={() => copyText(decodeHtml(jsxSlot.firstChild.innerHTML))}>
              {#if isClipboardButtonPressed}
                <svg
                  class="h-5 w-5 fill-current"
                  xmlns="http://www.w3.org/2000/svg"
                  viewBox="0 0 32 32">
                  <path
                    d="M 16 2 C 14.742188 2 13.847656 2.890625 13.40625 4 L 5 4 L 5 29 L 27 29 L 27 4 L 18.59375 4 C 18.152344 2.890625 17.257813 2 16 2 Z M 16 4 C 16.554688 4 17 4.445313 17 5 L 17 6 L 20 6 L 20 8 L 12 8 L 12 6 L 15 6 L 15 5 C 15 4.445313 15.445313 4 16 4 Z M 7 6 L 10 6 L 10 10 L 22 10 L 22 6 L 25 6 L 25 27 L 7 27 Z M 21.28125 13.28125 L 15 19.5625 L 11.71875 16.28125 L 10.28125 17.71875 L 14.28125 21.71875 L 15 22.40625 L 15.71875 21.71875 L 22.71875 14.71875 Z" />
                </svg>
              {:else}
                <svg
                  class="h-5 w-5 fill-current"
                  xmlns="http://www.w3.org/2000/svg"
                  viewBox="0 0 32 32">
                  <path
                    d="M 16 3 C 14.742188 3 13.847656 3.890625 13.40625 5 L 6 5 L 6 28 L 26 28 L 26 5 L 18.59375 5 C 18.152344 3.890625 17.257813 3 16 3 Z M 16 5 C 16.554688 5 17 5.445313 17 6 L 17 7 L 20 7 L 20 9 L 12 9 L 12 7 L 15 7 L 15 6 C 15 5.445313 15.445313 5 16 5 Z M 8 7 L 10 7 L 10 11 L 22 11 L 22 7 L 24 7 L 24 26 L 8 26 Z" />
                </svg>
              {/if}
            </button>
          </div>
        </div>
      </div>
    {/if}
  </div>
</div>

<style global lang="postcss">
  .prose .code-wrapper {
    max-width: 100vw;
    overflow-x: auto;
  }
  .prose .code-wrapper > pre {
    max-height: 32em;
  }
  .prose .component-preview pre[class*="language-"] {
    width: 100%;
    max-width: 100%;
    font-size: 14px;
    padding-right: 2.5rem;
    padding-top: 1rem;
    padding-bottom: 1rem;
    border-top-right-radius: var(--rounded-box, 1rem);
    border-top-left-radius: var(--rounded-box, 1rem);
    border-bottom-right-radius: var(--rounded-box, 1rem);
    border-bottom-left-radius: var(--rounded-box, 1rem);
    margin: 0;
    min-height: 6rem;
  }
  .prose .component-preview pre[class*="language-"] .token.comment {
    color: var(--fallback-nc, oklch(var(--nc) / 0.4));
  }
  .prose .component-preview .preview {
    background-image: repeating-linear-gradient(
      45deg,
      var(--fallback-b1, oklch(var(--b1))),
      var(--fallback-b1, oklch(var(--b1))) 13px,
      var(--fallback-b2, oklch(var(--b2))) 13px,
      var(--fallback-b2, oklch(var(--b2))) 14px
    );
    /* background-size: 40px 40px;
    background-image: linear-gradient(to right, oklch(var(--bc) / 0.04) 1px, transparent 1px),
      linear-gradient(to bottom, oklch(var(--bc) / 0.04) 1px, transparent 1px); */
  }
</style>
