<script>
  import { getContext, createEventDispatcher } from "svelte";
  import { ROUTER, LOCATION } from "./contexts.js";
  import { navigate } from "./history.js";
  import { startsWith, resolve, shouldNavigate } from "./utils.js";

  export let to = "#";
  export let replace = false;
  export let state = {};
  export let getProps = () => ({});

  const { base, hashed } = getContext(ROUTER);
  const location = getContext(LOCATION);
  const dispatch = createEventDispatcher();

  let href, isPartiallyCurrent, isCurrent, props;
  /**
   * href holds (base.uri + to) value to allow hashtag char (#)
   * if hashtag routing used
   */
  if (to === "/") {
    href = $base.uri;
  } else if (to.startsWith("#")) {
    href = $base.uri + to;
  } else
    href = resolve(to, $base.uri);
  $: isPartiallyCurrent = hashed ?
          startsWith($location.href, href) :
          startsWith($location.pathname, href);
  $: isCurrent = href === (
          hashed ?
                  $location.href :
                  $location.pathname
  );
  $: ariaCurrent = isCurrent ? "page" : undefined;
  $: props = getProps({
    location: $location,
    href,
    isPartiallyCurrent,
    isCurrent
  });

  function onClick(event) {
    dispatch("click", event);

    if (shouldNavigate(event)) {
      event.preventDefault();
      // Don't push another entry to the history stack when the user
      // clicks on a Link to the page they are currently on.
      const shouldReplace = $location.pathname === href || replace;
      navigate(href, { state, replace: shouldReplace });
    }
  }
</script>

<a href="{href}" aria-current="{ariaCurrent}" on:click="{onClick}" {...props} {...$$restProps}>
  <slot></slot>
</a>
