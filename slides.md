---
highlighter: shiki
layout: cover
lineNumbers: true
background: >-
  https://images.pexels.com/photos/2881229/pexels-photo-2881229.jpeg?auto=compress&cs=tinysrgb&dpr=2&h=750&w=1260
title: 🔌 Entkoppelung von Games im Frontend von {BOTolution}
---

# 🔌 Entkoppelung von Games im Frontend von {BOTolution}

Vortrag von Antonio Sarcevic

<!--
- Willkommen
- Antonio Sarcevic, zuständig für Frontend von {BOTolution}
- im Vortrag: Entkoppelung von Games (mit dazugehörigen Komponenten) im Frontend
-->

---

# Inhalt

- ⚙️ Game Parameters
- 🎲 Game UI Component

_jeweils ..._

1. 🏆 Ziele
1. 🏁 Ausgangsposition
1. 🧪 Technologie
1. 🔨 Umsetzung

<!--
- Zwei bereiche Entkoppelt
  - Game Parameters
    - Einstellen von spielspezifischen Parametern beim Erstellen von Matches / Turnieren
    - z.B. TicTacToe: Spielfeld Größe und benötigte Länge zum Sieg.
  - Game UI Component
    - Anzeigen des spielspezifischen Zustands
    - z.B. TicTacToe: Spielfeld mit Kreuzen und Kreisen

- Jeweils Einblick in ...
-->

---

# ⚙️ Game Parameters

🏆 Ziele

<div class="grid grid-cols-2 gap-4">

- 📝 Formularelemente
- ✏️ `matchConfig` mutieren
- 🔢 Standardwerte
- 🔌 Entkoppelung 🆕

<img class="mx-auto" src="/parameters.png" alt="Game Parameters Form">

</div>

<!--
- dynamische Formularelemente für jedes Spiel spezifisch
- Anpassung matchConfig welche zur Initialisierung ans Backend geschickt wird
- Standardwerte für jedes Feld oder sogar Min Max values
- jetzt neu: Entkoppelung von Platform und Spiel
  - Frontend sollte nicht neu gebaut werden für neues Spies
 -->

---

# ⚙️ Game Parameters

[🏁 Ausgangsposition: `TicTacToeParameters.svelte`](https://git.fh-muenster.de/swa1/coding-challenge/platform/-/blob/987a29b89ea68bdb56037922bf41bf73560ca667/FrontEnd/src/lib/games/TicTacToeParameters.svelte)

```html{all|4-13|15-19|23-47}
<script>
  import { onMount } from "svelte";

  // 🔢 Standardwerte ✅
  let defaults = {
    parameters: {
      gameBoardSize: {
        rows: 3,
        columns: 3,
        winningLength: 3,
      },
    },
  };

  // ✏️ `matchConfig` mutieren ✅
  export let matchConfig;
  onMount(() => {
    matchConfig = { ...matchConfig, ...defaults };
  });
</script>

{#if matchConfig.parameters && matchConfig.parameters.gameBoardSize}
<!-- 📝 Formularelemente ✅ -->
<label>
  # of rows
  <input
    type="number"
    min="1"
    bind:value="{matchConfig.parameters.gameBoardSize.rows}"
  />
</label>
<label>
  # of columns
  <input
    type="number"
    min="1"
    bind:value="{matchConfig.parameters.gameBoardSize.columns}"
  />
</label>
<label>
  Length of connected tiles to win
  <input
    type="number"
    min="1"
    bind:value="{matchConfig.parameters.gameBoardSize.winningLength}"
  />
</label>
{/if}
```

<!--
- Disclaimer: Code zur veranschaulichung angepasst

- Bisherige Lösung für Game Parameter

- Für jedes Game eine GameIdParameter Komponente die Ziele erfüllt

  - Standardwerte
    - in Variable definiert
  - matchConfig mutieren
    - props durch export
    - anwenden der Defaults onMount
  - Formularelemente
    - Input Felder mit bind: mutieren matchConfig

-->

---

# ⚙️ Game Parameters

[🏁 Ausgangsposition: `NewMatch.svelte`](https://git.fh-muenster.de/swa1/coding-challenge/platform/-/blob/987a29b89ea68bdb56037922bf41bf73560ca667/FrontEnd/src/pages/matches/_components/NewMatch.svelte)

```html{all|22-27}
<script>
  import TicTacToeParameters from "./TicTacToeParameters.svelte";
  import UnoParameters from "./UnoParameters.svelte";

  let games = [{ id: "tictactoe" }, { id: "uno" }];

  let matchConfig = {
    gameId: "",
  };
</script>

<form on:submit>
  <label>
    Game
    <select labelText="Game" bind:value="{matchConfig.gameId}">
      {#each games as game}
      <option value="{game.id}">{game.id}</option>
      {/each}
    </select>
  </label>

  <!-- 🔌 Entkoppelung ❌ -->
  {#if matchConfig.gameId === "tictactoe"}
  <TicTacToeParameters bind:matchConfig />
  {:else if matchConfig.gameId === "uno"}
  <UnoParameters bind:matchConfig />
  {/if}
</form>

<style>
  form {
    display: grid;
    gap: 1em;
  }
  :global(label) {
    display: grid;
  }
</style>
```

<!--
- NewMatch.svelte regelt das erstellen für ein neues Match
- Importiert alle Spiele Parameter Komponenten
  - zeigt abhängig von gewähltem Spiel an
- Entkoppelung nicht gegeben
  - Spiele händisch zu dem wachsendem If/Else Block hinzugefügt werden
 -->

---

# ⚙️ Game Parameters

🧪 Technologie

<img src="/json-schema-logo.png" alt="JSON Schema Logo" class="absolute top-10 right-10 w-100">

- GameParameters: Beschreibung eines JSON Objekts
- JSON Schema: Sprache zur Beschreibung JSON Objekte

```json
{} // erlaubt valides JSON jeder Art
```

```json
{ "type": "string" } // erlaubt JSON vom Typ String (z.B. "Hello JSON Schema")
```

```json
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "type": "object",
  "properties": {
    "someKey": {
      "type": "string"
    }
  }
} // erlaubt ein JSON Objekt mit someKey und Wert vom Typ String (z.B. { someKey: "Hello JSON Schema" })
```

<!--
- Game Parameters als beschreibung eines JSON Objekts
- JSON Schema als Spezifikation von JSON Dokumenten
  - kann z.B. Objekte Beschreiben
  - wird in der Regel zum validieren von JSON Dokumenten genutzt
 -->

---

# ⚙️ Game Parameters

🧪 Technologie in Bezug auf 🏆 Ziele

- 🔌 Entkoppelung durch JSON Endpunkt ✅
- 🔢 JSON Schema unterstützt Standardwerte ✅

  ```json
  {
    "$schema": "http://json-schema.org/draft-04/schema#",
    "type": "object",
    "properties": {
      "someKey": {
        "type": "string",
        "default": "default value"
      }
    }
  }
  ```

<!--
- Bietet Möglichkeit Default, Min und Max werte anzugeben

- Schema ist JSON an sich
  - Kann entkoppelt von einem beliebigen Endpunkt zur laufzeit geladen werden
 -->

---

# ⚙️ Game Parameters

🧪 Technologie

- 📝 Formularelemente 👀

  --> Frontendseitiges interpretieren der JSON Schema notwendig

- ✏️ `matchConfig` mutieren 👀

  --> Frontendseitiges verknüpfen der Input Elemente mit `matchConfig` Objekt

<!--

- kein Paket zum parsen und erstellen von Formularelementen + Mutieren von Objekten

-->

---

# ⚙️ Game Parameters

[🔨 Umsetzung: `/games/tictactoe/parameters.json`:](https://git.fh-muenster.de/swa1/coding-challenge/games/cc-tictactoe/-/blob/master/parameters.json)

```json
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "type": "object",
  "properties": {
    "gameBoardSize": {
      "type": "object",
      "properties": {
        "rows": {
          "type": "number",
          "default": 3,
          "minimum": 1
        },
        "columns": {
          "type": "number",
          "default": 3,
          "minimum": 1
        },
        "winningLength": {
          "type": "number",
          "default": 3,
          "minimum": 1
        }
      }
    }
  }
}
```

<!--
- JSON Schema für TicTacToe GameParameters
- Ausgeliefert über Microservice Endpunkt bzw. Referee Container
- JSON Schema kann auch minimum und maximum!
-->

---

# ⚙️ Game Parameters

[🔨 Umsetzung: `GamePropField.svelte`:](https://git.fh-muenster.de/swa1/coding-challenge/platform/-/blob/master/FrontEnd/src/lib/games/GameParameters/GamePropField.svelte)

<!-- prettier-ignore-start -->
```html
<script>
  import titleCase from "../../utils/titleCase";
  import { createEventDispatcher, tick } from "svelte";

  const dispatch = createEventDispatcher();

  export let props = {};
  export let name = "";
  let displayName = titleCase(name);

  async function handleFieldChange() {
    await tick();
    dispatch("bubbleUpChange", { path: [name], value });
  }

  function handleBubbleUpChange(evt) {
    if (name) {
      let newPath = [name, ...evt.detail.path];
      dispatch("bubbleUpChange", { path: newPath, value: evt.detail.value });
    } else {
      dispatch("bubbleUpChange", evt.detail);
    }
  }

  let value = props.default;
  if (name) {
    handleFieldChange();
  }
</script>

<div class="field">
  {#if props.type === "object"}
    {#if displayName}
      {displayName}
    {:else}
      Game Parameters
    {/if}
    {#each Object.keys(props.properties) as prop}
      <div class="child">
        <svelte:self
          props="{props.properties[prop]}"
          name="{prop}"
          on:bubbleUpChange="{handleBubbleUpChange}"
        />
      </div>
    {/each}
  {:else if props.type === "number"}
    <label>
      {displayName}
      <input
        type="number"
        min="{props.minimum}"
        max="{props.maximum}"
        bind:value
        on:change="{handleFieldChange}"
      />
    </label>
  {:else if props.type === "string"}
    <label>
      {displayName}
      <input type="text" bind:value on:change="{handleFieldChange}" />
    </label>
  {:else}
    {name} # undefined type
  {/if}
</div>

<style>
  .field {
    margin: 8px 0;
  }
  .child {
    margin-left: 8px;
  }
</style>
```
<!-- prettier-ignore-end -->

<!-- asdfasfd -->

---

# ⚙️ Game Parameters

[🔨 Umsetzung: `GameParameters.svelte`:](https://git.fh-muenster.de/swa1/coding-challenge/platform/-/blob/master/FrontEnd/src/lib/games/GameParameters/GameParameters.svelte)

```html
<script>
  import { InlineNotification, Loading } from "carbon-components-svelte";
  import { onMount } from "svelte";
  import GamePropField from "./GamePropField.svelte";

  export let config;

  onMount(() => {
    config = { ...config, parameters: {} };
  });

  async function getParamSchema(gameId) {
    const res = await fetch(`/games/${gameId.toLowerCase()}/parameters.json`);
    const json = await res.json();
    if (res.ok) {
      return json;
    } else {
      throw new Error(json);
    }
  }

  let promise = getParamSchema(config.gameId);

  function handleBubbleUpChange(evt) {
    setValueWithPath(evt.detail.value, evt.detail.path);
  }

  function setValueWithPath(value, path) {
    let schema; // a moving reference to internal objects within obj
    if ("gameParameters" in config) {
      schema = config.gameParameters; // TournamentConfig
    } else if ("parameters" in config) {
      schema = config.parameters; // MatchConfig
    }
    let len = path.length;
    for (var i = 0; i < len - 1; i++) {
      let elem = path[i];
      if (!schema[elem]) schema[elem] = {};
      schema = schema[elem];
    }

    schema[path[len - 1]] = value;

    config = config;
  }
</script>

{#await promise}
<Loading withOverlay="{false}" />
{:then props}
<GamePropField {props} on:bubbleUpChange="{handleBubbleUpChange}" />
{:catch _}
<InlineNotification
  title="Fehler"
  subtitle="Entschuldige, irgendwas ist bei uns schief gelaufen."
/>
{/await}
```

<!-- asdfasfd -->

---

Files before Commit: https://git.fh-muenster.de/swa1/coding-challenge/platform/-/tree/987a29b89ea68bdb56037922bf41bf73560ca667
Commit: https://git.fh-muenster.de/swa1/coding-challenge/platform/-/commit/417eacc1005ab37b60c52a8ebc4a5564d9c6e549
Files with Commit: https://git.fh-muenster.de/swa1/coding-challenge/platform/-/tree/417eacc1005ab37b60c52a8ebc4a5564d9c6e549

# 🎲 Game UI Component

Files before Commit: https://git.fh-muenster.de/swa1/coding-challenge/platform/-/tree/8e8d0dc48a0a4b2a39dafcf5d6dccdca83f44fbc
Commit: https://git.fh-muenster.de/swa1/coding-challenge/platform/-/commit/62ce4c62cabc0c7434fcec94a412179e045ee969
Files with Commit: https://git.fh-muenster.de/swa1/coding-challenge/platform/-/tree/62ce4c62cabc0c7434fcec94a412179e045ee969
