<script lang="ts">
  import "$lib/style.css";
  import { circOut } from "svelte/easing";
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
    let currentSign = numberNegative ? "-" : "";
    let decimal = usingDecimal ? "." : "";
    let zeros = "";

    if (zerosAdded > 0) {
      zeros = new Array(zerosAdded)
        .fill(null)
        .map(() => "0")
        .reduce((previous, current) => previous + current);
    }

    if (currentOperation && secondNumber?.toString().includes(".")) {
      decimal = "";
    }

    if (!currentOperation && firstNumber?.toString().includes(".")) {
      decimal = "";
    }

    if (secondNumber || currentOperation) {
      currentDisplay =
        currentSign + (secondNumber || 0).toString() + decimal + zeros;

      return;
    }

    currentDisplay =
      currentSign + (firstNumber || 0).toString() + decimal + zeros;
  });

  // methods
  function toggleNumberNegative() {
    numberNegative = !numberNegative;
  }

  function toggleUsingDecimal() {
    if (currentOperation && secondNumber?.toString().includes(".")) {
      return;
    }

    if (!currentOperation && firstNumber?.toString().includes(".")) {
      return;
    }

    usingDecimal = !usingDecimal;
  }

  function appendNumber(number: number) {
    if (currentOperation !== null) {
      if (secondNumber && secondNumber.toString().length > 10) {
        return;
      }

      let secondNumberHasDecimal = secondNumber?.toString().includes(".");

      if (usingDecimal || secondNumberHasDecimal) {
        if (number === 0) {
          zerosAdded++;

          return;
        }

        secondNumber =
          (secondNumber || 0) +
          number /
            (secondNumber !== null &&
            secondNumber.toString().split(".").length > 1
              ? 10 **
                (secondNumber.toString().split(".")[1].length +
                  1 +
                  zerosAdded +
                  1)
              : 10 ** (zerosAdded + 1));

        zerosAdded = 0;

        return;
      }

      secondNumber = (secondNumber || 0) * 10 + number;

      return;
    }

    if (firstNumber && firstNumber.toString().length > 10) {
      return;
    }

    let firstNumberHasDecimal = firstNumber?.toString().includes(".");

    if (usingDecimal || firstNumberHasDecimal) {
      if (number === 0) {
        zerosAdded++;

        return;
      }

      firstNumber =
        (firstNumber || 0) +
        number /
          (firstNumber !== null && firstNumber.toString().split(".").length > 1
            ? 10 **
              (firstNumber.toString().split(".")[1].length + zerosAdded + 1)
            : 10 ** (zerosAdded + 1));

      zerosAdded = 0;

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
    submitCurrentNumber();

    currentOperation = operation;
    clearOperands();
  }

  function submitCurrentNumber() {
    if (!currentOperation) {
      if (numberNegative && firstNumber !== null) {
        firstNumber = -firstNumber;
      }

      return;
    }
  }

  function getResult() {
    if (!currentOperation) {
      return;
    }

    let localResults = 0;

    if (firstNumber === null) {
      firstNumber = 0;
    }

    if (secondNumber === null) {
      secondNumber = firstNumber;
    }

    if (numberNegative) {
      secondNumber = -secondNumber;
    }

    switch (currentOperation) {
      case "div":
        localResults = firstNumber / secondNumber;
        break;
      case "add":
        localResults = firstNumber + secondNumber;
        break;
      case "mul":
        localResults = firstNumber * secondNumber;
        break;
      case "sub":
        localResults = firstNumber - secondNumber;
        break;
    }

    hardClear();
    firstNumber = localResults;
  }

  // calculator flying animation
  let showCalculator = $state(false);

  $effect(() => {
    showCalculator = true;
  });
</script>

<div class="calculator-container">
  {#if showCalculator}
    <div
      class="calculator"
      in:fly={{
        y: "150%",
        easing: circOut,
        duration: 400,
        delay: 100,
      }}
    >
      <div class="display">{currentDisplay}</div>

      <div class="grid input-grid">
        <div class="grid operands-grid">
          <button on:click={clear} class="tertiary-button"
            >{firstNumber === null && secondNumber === null
              ? "C"
              : "AC"}</button
          >
          <button class="tertiary-button" on:click={toggleNumberNegative}>
            +/-
          </button>
          <button class="tertiary-button" on:click={toggleUsingDecimal}>
            .
          </button>

          {#each new Array(10).fill(null).map((_, i) => 9 - i) as number}
            <button
              on:click={() => appendNumber(number)}
              class:span-3={number === 0}>{number}</button
            >
          {/each}
        </div>

        <div class="grid operations-grid">
          {#each operations as operation, i}
            <button
              on:click={() => setCurrentOperation(operation)}
              class="secondary-button"
              class:active-button={currentOperation === operation}
            >
              {@html operationsDisplay[i]}
            </button>
          {/each}
          <button on:click={getResult} class="accent-button">=</button>
        </div>
      </div>
    </div>
  {/if}
</div>

<style>
  button {
    height: fit-content;
    padding: 1rem;
    background-color: rgba(255, 255, 255, 0.69);
    border-radius: 0.5rem;
    box-shadow: 0 4px 30px rgba(0, 0, 0, 0.1);
    backdrop-filter: blur(4px);
    -webkit-backdrop-filter: blur(4px);
    border: none;
    font-size: 2rem;
    cursor: pointer;
    transition: 200ms ease-in-out;
    transition-property: transform;
  }

  button:hover {
    transform: translateY(-3px);
    opacity: 0.8;
  }

  button:active {
    opacity: 0.6;
  }

  .secondary-button {
    background-color: rgb(255, 157, 0, 0.7);
    color: white;
  }

  .tertiary-button {
    background-color: rgba(104, 104, 104, 0.7);
    color: white;
  }

  .accent-button {
    background-color: rgba(255, 30, 0, 0.7);
    color: white;
  }

  .active-button {
    background-color: rgba(207, 130, 5, 0.7);
  }

  .calculator-container {
    width: 100vw;
    height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .calculator {
    padding: 1rem;
    background: rgba(255, 255, 255, 0.7);
    border-radius: 1rem;
    box-shadow: 0 4px 30px rgba(0, 0, 0, 0.1);
    backdrop-filter: blur(4px);
    -webkit-backdrop-filter: blur(4px);
    display: flex;
    gap: 2rem;
    flex-direction: column;
  }

  .display {
    padding: 1rem;
    background: rgba(255, 255, 255, 0.9);
    border-radius: 0.5rem;
    box-shadow: 0 4px 30px rgba(0, 0, 0, 0.1);
    backdrop-filter: blur(4px);
    -webkit-backdrop-filter: blur(4px);
    font-size: 2.5rem;
    text-align: right;
  }

  .grid {
    display: grid;
    gap: 1rem;
  }

  .input-grid {
    grid-template-columns: repeat(4, 1fr);
  }

  .operands-grid {
    grid-column: span 3;
    grid-template-columns: repeat(3, 1fr);
  }

  .span-3 {
    grid-column: span 3;
  }
</style>
