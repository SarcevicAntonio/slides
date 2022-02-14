---
highlighter: shiki
layout: cover
lineNumbers: true
background: https://images.pexels.com/photos/2881229/pexels-photo-2881229.jpeg?auto=compress&cs=tinysrgb&dpr=2&h=750&w=1260
title: 🔌 Entkoppelung von Games im Frontend von {BOTolution}
---

# 🔌 Entkoppelung von Games im Frontend von {BOTolution}

Vortrag von Antonio Sarcevic - Masterprojekt: Coding Challenge / {BOTolution} - SS 2021 + WS 2021/22

<img class="mt-4" src="/fh.svg" alt="FH Münster Logo">

<!-- prettier-ignore-start -->
<!--
- Willkommen
- Antonio Sarcevic, zuständig für Frontend von {BOTolution}
- im Vortrag: Entkoppelung von Games (mit dazugehörigen Komponenten) im Frontend
-->
---
layout: image-right
image: https://images.unsplash.com/photo-1610056494052-6a4f83a8368c?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=1287&q=80
---
<!-- prettier-ignore-end -->

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

## 🏆 Ziele

- 📝 Formularelemente
- ✏️ `matchConfig` mutieren
- 🔢 Standardwerte
- 🔌 Entkoppelung 🆕

<img class="hover-right" src="/parameters.png" alt="Game Parameters Form">

<!--
- dynamische Formularelemente für jedes Spiel spezifisch
- Anpassung matchConfig welche zur Initialisierung ans Backend geschickt wird
- Standardwerte für jedes Feld oder sogar Min Max values
- jetzt neu: Entkoppelung von Platform und Spiel
  - Frontend sollte nicht neu gebaut werden für neues Spies
-->

---

# ⚙️ Game Parameters

## [🏁 Ausgangsposition: `TicTacToeParameters.svelte`](https://git.fh-muenster.de/swa1/coding-challenge/platform/-/blob/987a29b89ea68bdb56037922bf41bf73560ca667/FrontEnd/src/lib/games/TicTacToeParameters.svelte)

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

## [🏁 Ausgangsposition: `NewMatch.svelte`](https://git.fh-muenster.de/swa1/coding-challenge/platform/-/blob/987a29b89ea68bdb56037922bf41bf73560ca667/FrontEnd/src/pages/matches/_components/NewMatch.svelte)

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

## 🧪 Technologie

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

## 🧪 Technologie in Bezug auf 🏆 Ziele

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
- Schema ist JSON an sich
  - Kann entkoppelt von einem beliebigen Endpunkt zur laufzeit geladen werden

- Bietet Möglichkeit Default, Min und Max werte anzugeben
-->

---

# ⚙️ Game Parameters

## 🧪 Technologie in Bezug auf 🏆 Ziele

- 📝 Formularelemente 👀

  --> Frontendseitiges interpretieren der JSON Schema notwendig

- ✏️ `matchConfig` mutieren 👀

  --> Frontendseitiges verknüpfen der Input Elemente mit `matchConfig` Objekt

<!--

- kein Paket zum parsen und erstellen von Formularelementen + Mutieren von Objekten

-->

---

# ⚙️ Game Parameters

## [🔨 Umsetzung: `tictactoe/parameters.json`](https://git.fh-muenster.de/swa1/coding-challenge/games/cc-tictactoe/-/blob/master/parameters.json)

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
- Hosten der JSON Datei über Endpunkt
-->

---

# ⚙️ Game Parameters

## 🔨 Umsetzung: TODO

- 🔌 Passendes parameters.json zum Game fetchen
- 📝 Rendern der passenden Formularelemente
- ✏️ Binden der Formularelemente an das `matchConfig` Objekt

---

# ⚙️ Game Parameters

## [🔨 Umsetzung: `GameParameters.svelte`](https://git.fh-muenster.de/swa1/coding-challenge/platform/-/blob/master/FrontEnd/src/lib/games/GameParameters/GameParameters.svelte)

```html{all|12-21|53-54|25-47}
<script>
  import { InlineNotification, Loading } from "carbon-components-svelte";
  import { onMount } from "svelte";
  import GamePropField from "./GamePropField.svelte";

  export let config;

  onMount(() => {
    config = { ...config, parameters: {} };
  });

  // 🔌 Passendes parameters.json zum Game fetchen
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

  // ✏️ Binden der Formularelemente an das `matchConfig` Objekt
  function handleBubbleUpChange(evt) {
    setValueWithPath(evt.detail.value, evt.detail.path);
  }

  function setValueWithPath(value, path) {
    let schema;
    if ("gameParameters" in config) {
      schema = config.gameParameters;
    } else if ("parameters" in config) {
      schema = config.parameters;
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
<!-- 📝 Rendern der passenden Formularelemente -->
<GamePropField {props} on:bubbleUpChange="{handleBubbleUpChange}" />
{:catch _}
<InlineNotification
  title="Fehler"
  subtitle="Entschuldige, irgendwas ist bei uns schief gelaufen."
/>
{/await}
```

<!--
1. RUN:
- Passendes parameters.json zum Game fetchen
- Aufrufen von GamePropField und passen des JSON Schemas

2. RUN: handleBubbleUpChange
- Setzen des `config` Objekts mittels des Paths aus dem Event.
- handleBubbleUpChange -> setValueWithPath
  - Nutzen des Path Arrays um das Objekt zu mutieren
  - tbh weiß ich nicht mehr genau wie das klappt :D
 -->

---

# ⚙️ Game Parameters

## [🔨 Umsetzung: `GamePropField.svelte`](https://git.fh-muenster.de/swa1/coding-challenge/platform/-/blob/master/FrontEnd/src/lib/games/GameParameters/GamePropField.svelte)

<!-- prettier-ignore-start -->
```html{all|34-73|11-26}
<script>
  import titleCase from "../../utils/titleCase";
  import { createEventDispatcher, tick } from "svelte";

  const dispatch = createEventDispatcher();

  export let props = {};
  export let name = "";
  let displayName = titleCase(name);

  // ✏️ Binden der Formularelemente an das `matchConfig` Objekt
  async function handleFieldChange() {
    await tick();
    // Senden eines bubbleUpChange Events
    dispatch("bubbleUpChange", { path: [name], value });
  }

  // Hören der rekursiven Aufrufe von bubbleUpChange
  function handleBubbleUpChange(evt) {
    if (name) {
      let newPath = [name, ...evt.detail.path];
      dispatch("bubbleUpChange", { path: newPath, value: evt.detail.value });
    } else { // Dem initialen Aufruf wird kein Name gegeben, dieser leitet nur weiter.
      dispatch("bubbleUpChange", evt.detail);
    }
  }

  let value = props.default;
  if (name) {
    handleFieldChange();
  }
</script>

<!-- 📝 Rendern der passenden Formularelemente -->
<div class="field">
  {#if props.type === "object"}
    {#if displayName}
      {displayName}
    {:else}
      Game Parameters
    {/if}
    <!-- Object: Rekursives aufrufen der Komponente mit allen Properties -->
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
    <!-- Number: Rendern eines Number Inputs -->
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
    <!-- String: Rendern eines Text Inputs -->
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

<!--
1. RUN:
- Rendern der passenden Formularelemente
  - Unterscheidung zwischen Objekt, Number und String

2. RUN start:

- on:change -> handleFieldChange
  - Senden eines bubbleUpChange Events; Name des Keys wird übergeben um einen Path zu bilden
  - Bei Rekursive / Geschachtelte Objekt keys wird mit handleBubbleUpChange der Path zusammengebaut
    - Initiale Aufruf kein Name also wird nur weitergeleitet
 -->

---

# 🎲 Game UI Component

## 🏆 Ziele

- 👀 Game UI
- ➡️📦 Daten Input
- ⬅️📦 Daten Output
- 🔌 Entkoppelung 🆕

<img class="hover-right" src="/ui-comp.png" alt="Game Watch Screen">

<!--
- dynamische Game UI für jedes Spiel spezifisch
- Annehmen von Daten z.B. Game initialization und move to draw
- Ausgeben von spielspezifischen Zugbeschreibungen
- jetzt neu: Entkoppelung von Platform und Spiel
  - Frontend sollte nicht neu gebaut werden für neues Spies
 -->

---

# 🎲 Game UI Component

## [🏁 Ausgangsposition: `TicTacToe.svelte`](https://git.fh-muenster.de/swa1/coding-challenge/platform/-/blob/8e8d0dc48a0a4b2a39dafcf5d6dccdca83f44fbc/FrontEnd/src/lib/games/TicTacToe.svelte)

```html{all|6-8|21-25|39-55}
<script>
  import { afterUpdate, createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();

  // ➡️📦 Daten Input ✅
  export let initialization = { gameBoard: [] };
  export let drawMove;

  let gameBoard = initialization.gameBoard;


  afterUpdate(() => {
    let turn = drawMove.turn;
    if (turn) {
      switch (drawMove.flag) {
        case "do":
          // Zeichne token
          gameBoard[turn.move.row][turn.move.column] = turn.move.botToken;
          // sende matchState Message
          // ⬅️📦 Daten Output ✅
          dispatch(
            "moveDrawn",
            `Token "${turn.move.botToken}" was placed in row ${turn.move.row} and column ${turn.move.column}`
          );
          break;
        case "undo":
          // lösche token
          gameBoard[turn.move.row][turn.move.column] = "";
          break;
        default:
          // match just initalized
          break;
      }
    }
  });
</script>

<!-- 👀 Game UI ✅ -->
<div class="field">
  {#each gameBoard as row}
    <div class="row">
      {#each row as tile}
        <div class="tile">
          {#if tile === "O"}
            ⭕
          {/if}
          {#if tile === "X"}
            ❌
          {/if}
        </div>
      {/each}
    </div>
  {/each}
</div>

<style>
  .field {
    margin: 0 auto;
    padding: 16px;
    display: inline-block;
    background-color: var(--cds-ui-01, #f4f4f4);
  }
  .row {
    display: flex;
  }
  .tile {
    margin: 4px;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 40px;
    width: 40px;
    background-color: var(--cds-ui-05, #161616);
  }
</style>
```

<!--
- Disclaimer: Code zur veranschaulichung angepasst

- Bisherige Lösung für Game UI

- Für jedes Game eine Game.Svelte Komponente die Ziele erfüllt

  - Daten Input
    - über export als Svelte Prop definiert welche von außen befüllt werden kann
    - afterUpdate erlaubt auf Änderungen zu reagieren
  - Daten Output
    - dispatch um Spielspezifische Nachrichten zurück zu übertragen
  - Game UI
    - Anzeigen des Spielstatus über HTML
     -->

---

# 🎲 Game UI Component

## [🏁 Ausgangsposition: `GameRunner.svelte`](https://git.fh-muenster.de/swa1/coding-challenge/platform/-/blob/8e8d0dc48a0a4b2a39dafcf5d6dccdca83f44fbc/FrontEnd/src/pages/matches/%5BmatchId%5D/_components/GameRunner.svelte)

```html{all|74-87}
<script>
  import TicTacToe from "@lib/games/TicTacToe.svelte";
  import Uno from "@lib/games/Uno.svelte";
  import MatchTile from "@lib/MatchTile.svelte";
  import { afterUpdate } from "svelte";
  import { fade } from "svelte/transition";

  export let match;

  let matchStateHistory = [];

  let moveToDraw = { turn: {}, flag: "initital" };

  let turnIndex = -1;

  $: winner = match.turns[turnIndex]?.winner;

  function nextTurn() {
    turnIndex++;
    moveToDraw = { turn: match.turns[turnIndex], flag: "do" };
  }

  function previousTurn() {
    moveToDraw = { turn: match.turns[turnIndex], flag: "undo" };
    matchStateHistory.pop();
    matchStateHistory = matchStateHistory;
    turnIndex--;
  }

  $: previousTurnDisabled = turnIndex < 0;
  $: nextTurnDisabled = turnIndex >= match.turns.length - 1;

  /** handles keyboard shortcuts */
  function handleKeydown(e) {
    let key = e.key.toLowerCase();
    if (key === "j" && !previousTurnDisabled) {
      previousTurn(); // j
    } else if (key === "k" && !nextTurnDisabled) {
      nextTurn(); // k
    }
  }

  function saveMatchStateMessage(event) {
    matchStateHistory = [...matchStateHistory, event.detail];
  }

  let historyDiv;

  afterUpdate(() => {
    setTimeout(() => {
      historyDiv.scrollTo(0, historyDiv.scrollHeight);
    }, 1); // werid hack
  });
</script>

<svelte:window on:keydown="{handleKeydown}" />

<MatchTile {match} watchButton="{false}" />

<div class="row">
  <div>Turn #{turnIndex}</div>
  <div>
    <button on:click="{previousTurn}" disabled="{previousTurnDisabled}">
      ⬅️ Undo last turn (J)
    </button>
    <button on:click="{nextTurn}" disabled="{nextTurnDisabled}">
      ➡️ Do next turn (K)
    </button>
  </div>
</div>

<div class="row">
  <div>
    <!-- 🔌 Entkoppelung ❌ -->
    {#if match.gameId === "tictactoe"}
    <TicTacToe
      initialization="{match.initialization}"
      drawMove="{moveToDraw}"
      on:moveDrawn="{saveMatchStateMessage}"
    />
    {:else if match.gameId === "uno"}
    <Uno
      initialization="{match.initialization}"
      drawMove="{moveToDraw}"
      on:moveDrawn="{saveMatchStateMessage}"
    />
    {/if}
  </div>
  <div>
    <div class="history" bind:this="{historyDiv}">
      <ol start="0">
        {#each matchStateHistory as stateChange, index}
        <li transition:fade|local data-testid="matchStateHistoryMessage">
          Turn #{index}: {stateChange}
        </li>
        {/each}
      </ol>
      {#if winner}
      <div transition:fade|local>
        <span>{winner === "Draw" ? "Everybody" : winner} wins!</span>
        <span>GG WP! 👏🥳</span>
      </div>
      {/if}
    </div>
  </div>
</div>

<style>
  .history {
    overflow-y: auto;
  }
</style>
```

<!--
- GameRunner.svelte regelt das anzeigen eines Matches mit Step control etc
- Importiert alle Spiele UI Komponenten
  - zeigt abhängig von gewähltem Spiel an
- Entkoppelung nicht gegeben
  - Spiele händisch zu dem wachsendem If/Else Block hinzugefügt werden
 -->

---

# 🎲 Game UI Component

## 🧪 Technologie

<img src="/web-comps.png" alt="Web Component Components" class="absolute top-10 right-10 w-100">

- Game UI: Komponente
- Web Components: standardisiertes Komponenten Modell

  - Custom Elements
  - Shadow DOM
  - HTML Template

- Mehr über Web Components unter https://www.sarcevic.dev/blog/intro-to-webcomponents

- Svelte Compiler bietet Web Components als Output an

<!--
- Game UI als Komponente
  - isolierter Bestandteil
  - kann mit verschiedenem Inhalt gefüllt werden
  - kann mit Events nach außen Kommunizieren
- Web Components als standardisiertes Komponenten Modell für offenes Web
  - Ziel: unabhängig von Browsern und Frameworks Komponenten erstellen und verwenden + Hohe Flexibilität
  - besteht aus einer Reihe von APIs
    - Custom Elements: Erstellen von neuen HTML Tags bzw. DOM Elementen
    - Shadow DOM: Kapseln von Struktur, Style und Verhalten um z.B. Konflikte zu vermeiden
    - HTML Template: Definieren von wiederverwendbaren Dokumentfragmenten

- Letztes Jahr: Seminar Informatik Artikel über WebComponents hier Link als Blogpost (englisch)

- Svelte Compiler Output: Mitnahme des Alten Codes!
-->

---

# 🎲 Game UI Component

## 🧪 Technologie in Bezug auf 🏆 Ziele

- 👀 Game UI Svelte Code kann als Web Component kompiliert werden ✅
- ➡️📦 Daten Input über Attributes / Properties ✅
- ⬅️📦 Daten Output über Events ✅
- 🔌 Entkoppelung durch JS Endpunkt ✅

---

# 🎲 Game UI Component

## [🔨 Umsetzung: `TicTacToe.svelte`](https://git.fh-muenster.de/swa1/coding-challenge/games/cc-tictactoe/-/blob/master/gamefield/src/TicTacToe.svelte)

```html{all|1-2|8-15|19|all}
<!-- 👉 Definieren des HTMl Tags -->
<svelte:options tag="cc-tictactoe" />

<script>
  import { afterUpdate, createEventDispatcher } from "svelte";
  import { get_current_component } from "svelte/internal";

  // 👉 Svelte benötigt extra Boilerplate Code um Events von Web Components zu ermöglichen
  const component = get_current_component();
  const svelteDispatch = createEventDispatcher();
  const dispatch = (name, detail) => {
    svelteDispatch(name, detail);
    component.dispatchEvent &&
      component.dispatchEvent(new CustomEvent(name, { detail }));
  };

  // ➡️📦 Daten Input ✅
  export let initialization = { gameBoard: [] };
  export let move; /* 👉 HTML unterstützt nur kleingeschriebene Attributes */

  $: gameBoard = initialization.gameBoard;

  afterUpdate(() => {
    if (move?.turn) {
      let turn = move.turn;
      switch (move.flag) {
        case "do":
          if (turn.move && turn.isValid) {
            // Zeichne token
            gameBoard[turn.move.row][turn.move.column] = turn.move.botToken;
            // sende matchState Message
            // ⬅️📦 Daten Output ✅
            dispatch(
              "drawn",
              `Token "${turn.move.botToken}" was placed in row ${turn.move.row} and column ${turn.move.column}`
            );
          } else {
            dispatch("drawn", `Bot "${turn.move.timedOut}" timed out`);
          }

          break;
        case "undo":
          if (turn.move && turn.isValid) {
            // lösche token
            gameBoard[turn.move.row][turn.move.column] = "";
          }
          break;
        default:
          // match just initalized
          break;
      }
    }
  });
</script>

<!-- 👀 Game UI ✅ -->
<div class="field">
  {#each gameBoard as row}
    <div class="row">
      {#each row as tile}
        <div class="tile">
          {#if tile === "O"}
            ⭕
          {/if}
          {#if tile === "X"}
            ❌
          {/if}
        </div>
      {/each}
    </div>
  {/each}
</div>

<style>
  .field {
    margin: 0 auto;
    padding: 16px;
    display: inline-block;
    background-color: var(--cds-ui-01, #f4f4f4);
  }
  .row {
    display: flex;
  }
  .tile {
    margin: 4px;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 40px;
    width: 40px;
    background-color: var(--cds-ui-05, #161616);
  }
</style>
```

<!--
- Code weitestgehend übernommen, nur einige kleine Änderungen notwendig
  - Definieren des Custom Element HTML Tags über svelte:options
  - anpassen der Dispatch Funktion um etwas Boilerplate um Events von Web Components zu ermöglichen
  - anpassen von drawMove zu move, weil HTML Attribute klein geschrieben sein müssen
- Kompilieren der Svelte Datei zu einer JS Datei die Web Component definiert
- Hosting der JS Datei über Endpunkt!
-->

---

# 🎲 Game UI Component

## 🔨 Umsetzung: TODO

- 🔌 Passende Gamefield JS Datei importieren
- 👀 Game UI Web Component rendern
- ➡️📦 Daten Input von MatchRunner zur Web Component weiterleiten
- ⬅️📦 Daten Output von Web Component zum MatchRunner weiterleiten

---

# 🎲 Game UI Component

## [🔨 Umsetzung: `GamefieldWrapper.svelte`](https://git.fh-muenster.de/swa1/coding-challenge/platform/-/blob/master/FrontEnd/src/lib/matches/GamefieldWrapper.svelte)

```html{all|24-27|29-30,35,49|31-33,42-46|18-21,34,38-40|all}
<script>
  import {
    afterUpdate,
    createEventDispatcher,
    onDestroy,
    onMount,
  } from "svelte";

  const dispatch = createEventDispatcher();

  export let gameId;
  export let initialization;
  export let move;

  let wrapper;
  let field;

  // ⬅️📦 Daten Output von Web Component zum MatchRunner weiterleiten
  function forwardEvent(event) {
    dispatch("drawn", event.detail);
  }

  onMount(async () => {
    // 🔌 Passende Gamefield JS Datei importieren
    await import(
      /* @vite-ignore */ `/games/${gameId.toLowerCase()}/cc-${gameId.toLowerCase()}.js`
    );

    // 👀 Game UI Web Component rendern
    field = document.createElement("cc-" + gameId.toLowerCase());
    // ➡️📦 Daten Input von MatchRunner zur Web Component weiterleiten
    field["initialization"] = initialization;
    field["move"] = move;
    field.addEventListener("drawn", forwardEvent);
    wrapper.appendChild(field);
  });

  onDestroy(() => {
    field.removeEventListener("drawn", forwardEvent);
  });

  afterUpdate(() => {
    if (field && move) {
      field["move"] = move;
    }
  });
</script>

<div bind:this={wrapper}/>
```

<!-- prettier-ignore-start -->
<!--
- Passendes gamefield js vom Endpunkt über await import laden
- Rendern der Game UI Web Component: element erstellen in wrapper div appenden
- Daten Input: über Element Properties einmal initial und dann wenn sich move updated
- Daten Output: events werden weitergeleitet

- nutzen dieser Komponenten statt der Spielspezifischen Komponenten im MatchRunner.
 -->
---
layout: cover
background: https://images.pexels.com/photos/2881229/pexels-photo-2881229.jpeg?auto=compress&cs=tinysrgb&dpr=2&h=750&w=1260
---
<!-- prettier-ignore-end -->

# Vielen Dank für Ihre Aufmerksamkeit

Vortrag von Antonio Sarcevic - Masterprojekt: Coding Challenge / {BOTolution} - SS 2021 + WS 2021/22

<img class="mt-4" src="/fh.svg" alt="FH Münster Logo">
