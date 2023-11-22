<script lang="ts">
  import "$lib/style.css";
  import { fly } from "svelte/transition";

  const operations = ["div", "mul", "add", "sub"] as const;
  const operationsDisplay = ["&divide;", "&times;", "+", "-"] as const;

  let currentOperation = $state<(typeof operations)[number] | null>(null);
  let firstNumber = $state<number | null>(null);
  let secondNumber = $state<number | null>(null);

  // changes value of number when submitted
  let numberNegative = $state(false);
  let usingDecimal = $state(false);

  // only if the number has a decimal
  let zerosAdded = $state(0);

  // display on screen
  let currentDisplay = $state("");

  $effect(() => {
    if (secondNumber || currentOperation) {
      currentDisplay = (secondNumber || 0).toString();

      return;
    }

    currentDisplay = (firstNumber || 0).toString();
  });

  function appendNumber(number: number) {
    if (currentOperation !== null) {
      if (secondNumber && secondNumber.toString().length > 10) {
        return;
      }

      secondNumber = (secondNumber || 0) * 10 + number;

      return;
    }

    if (firstNumber && firstNumber.toString().length > 10) {
      return;
    }

    firstNumber = (firstNumber || 0) * 10 + number;
  }

  function clear() {
    if (secondNumber) {
      secondNumber = null;
      clearOperands();

      return;
    }

    if (currentOperation) {
      currentOperation = null;

      return;
    }

    firstNumber = null;
    clearOperands();
  }

  function hardClear() {
    clearOperands();
    secondNumber = null;
    firstNumber = null;
    currentOperation = null;
  }

  function clearOperands() {
    zerosAdded = 0;
    numberNegative = false;
    usingDecimal = false;
  }

  function setCurrentOperation(operation: (typeof operations)[number]) {
    currentOperation = operation;
    clearOperands();
  }

  function getResult() {
    if (!firstNumber || !currentOperation) {
      return;
    }

    let localResults = 0;

    switch (currentOperation) {
      case "div":
        if (secondNumber === null) {
          secondNumber = firstNumber;
        }
        localResults = firstNumber / secondNumber;
        break;
      case "add":
        if (secondNumber === null) {
          secondNumber = firstNumber;
        }
        localResults = firstNumber + secondNumber;
        break;
      case "mul":
        if (secondNumber === null) {
          secondNumber = firstNumber;
        }
        localResults = firstNumber * secondNumber;
        break;
      case "sub":
        if (secondNumber === null) {
          secondNumber = firstNumber;
        }
        localResults = firstNumber - secondNumber;
        break;
    }

    hardClear();
    firstNumber = localResults;
  }

  // calculator flying animation

  let mounted = false;

  $effect(() => {
    mounted = true;
  });
</script>

<div class="calculator-container">
  {#if mounted}
    <div class="calculator" in:fly>
      <div class="display">{currentDisplay}</div>

      <div class="grid">
        <div class="grid operands-grid">
          <button on:click={clear}>C</button>
          <button>+/-</button>
          <button>.</button>

          {#each new Array(10).fill(null).map((_, i) => 9 - i) as number}
            <button on:click={() => appendNumber(number)}>{number}</button>
          {/each}
        </div>

        <div class="grid operations-grid">
          {#each operations as operation, i}
            <button on:click={() => setCurrentOperation(operation)}>
              {@html operationsDisplay[i]}
            </button>
          {/each}
          <button on:click={getResult}>=</button>
        </div>
      </div>
    </div>
  {/if}
</div>

<style>
</style>
