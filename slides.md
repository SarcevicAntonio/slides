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

<br>

- ⚙️ Game Parameters
- 🎲 Game UI Component

<br>

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

<img class="mx-auto" src="/parameters.png" alt="Game Parameters Form">

- 📝 Formularelemente
- ✏️ `matchConfig` mutieren
- 🔢 Standardwerte
- 🔌 Entkoppelung 🆕

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

🏁 Ausgangsposition: `TicTacToeParameters.svelte`

```html{all|4-13|15-19|23-47}
<script>
  import { onMount } from "svelte";

  // 🔢 Standardwerte
  let defaults = {
    parameters: {
      gameBoardSize: {
        rows: 3,
        columns: 3,
        winningLength: 3,
      },
    },
  };

  // ✏️ `matchConfig` mutieren
  export let matchConfig;
  onMount(() => {
    matchConfig = { ...matchConfig, ...defaults };
  });
</script>

{#if matchConfig.parameters && matchConfig.parameters.gameBoardSize}
<!-- 📝 Formularelemente -->
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

🏁 Ausgangsposition: `NewMatch.svelte` _(gekürzt)_

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

# Game Parameters

🧪 Technologie

- GameParameters als beschreibung eines JSON Objekts
- JSON Schema um JSON Objekte zu beschreiben

```json
{} // erlaubt valides JSON jeder Art
```

```json
{ "type": "string" } // erlaubt JSON vom Typ String (z.B. "Hi")
```

```json
// erlaubt JSON Objekt mit someKey vom Typ String
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "type": "object",
  "properties": {
    "someKey": {
      "type": "string"
    }
  }
}
```

<!--
- Game Parameters als beschreibung eines JSON Objekts
- JSON Schema als Spezifikation von JSON Dokumenten
  - kann z.B. Objekte Beschreiben
  - wird in der Regel zum validieren von JSON Dokumenten genutzt
 -->

---

# Game Parameters

🧪 Technologie

- 🔢 Standardwerte ✅
- 🔌 Entkoppelung ✅

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

- 📝 Formularelemente 👀
- ✏️ `matchConfig` mutieren 👀

<!--
- Bietet Möglichkeit Default, Min und Max werte anzugeben

- Schema ist JSON an sich
  - Kann entkoppelt von einem beliebigen Endpunkt zur laufzeit geladen werden
 -->

---

# Game Parameters

Umsetzung

Files before Commit: https://git.fh-muenster.de/swa1/coding-challenge/platform/-/tree/987a29b89ea68bdb56037922bf41bf73560ca667
Commit: https://git.fh-muenster.de/swa1/coding-challenge/platform/-/commit/417eacc1005ab37b60c52a8ebc4a5564d9c6e549
Files with Commit: https://git.fh-muenster.de/swa1/coding-challenge/platform/-/tree/417eacc1005ab37b60c52a8ebc4a5564d9c6e549

# 🎲 Game UI Component

Files before Commit: https://git.fh-muenster.de/swa1/coding-challenge/platform/-/tree/8e8d0dc48a0a4b2a39dafcf5d6dccdca83f44fbc
Commit: https://git.fh-muenster.de/swa1/coding-challenge/platform/-/commit/62ce4c62cabc0c7434fcec94a412179e045ee969
Files with Commit: https://git.fh-muenster.de/swa1/coding-challenge/platform/-/tree/62ce4c62cabc0c7434fcec94a412179e045ee969
