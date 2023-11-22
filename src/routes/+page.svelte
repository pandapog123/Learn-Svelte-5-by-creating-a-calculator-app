<script lang="ts">
  import "$lib/style.css";
  import { circOut } from "svelte/easing";
  import { fly } from "svelte/transition";

  // calculator logic
  let currentEquation = $state<{
    firstNumber: number;
    secondNumber: number;
    result: number | null;
  }>({
    firstNumber: 0,
    secondNumber: 0,
    result: null,
  });

  let numberNegative = $state(false);
  let isAddingDecimal = $state(false);
  let zerosAdded = $state(0);
  let outputValue = $state("");
  let currentOperation = $state<"div" | "mul" | "add" | "sub" | null>(null);

  let canAddNumber = $derived(
    currentEquation.firstNumber.toString().length < 10
  );

  $effect(() => {
    let sign = numberNegative ? "-" : "";

    let decimal =
      isAddingDecimal &&
      !currentEquation.firstNumber
        .toString()
        .split("")
        .find((value) => value === ".")
        ? "."
        : "";

    if (currentOperation) {
      decimal =
        isAddingDecimal &&
        !currentEquation.secondNumber
          .toString()
          .split("")
          .find((value) => value === ".")
          ? "."
          : "";
    }

    let zeros = "";

    for (let i = 0; i < zerosAdded; i++) {
      zeros = zeros + "0";
    }

    outputValue = sign + currentEquation.firstNumber + decimal + zeros;
  });

  $effect(() => {
    if (currentOperation) {
      currentEquation.firstNumber = parseFloat(outputValue);
      isAddingDecimal = false;
      zerosAdded = 0;
      numberNegative = false;
    }
  });

  function insertNumber(number: number) {
    if (!canAddNumber) {
      return;
    }

    if (isAddingDecimal) {
      if (number === 0) {
        zerosAdded++;

        return;
      }

      let numberToAdd = number;

      for (
        let i = 0;
        i <
        (currentEquation.firstNumber.toString().split(".")[1]?.length + 1 ||
          1) +
          zerosAdded;
        i++
      ) {
        numberToAdd *= 0.1;
      }

      currentEquation.firstNumber = currentEquation.firstNumber + numberToAdd;
      zerosAdded = 0;
    } else {
      currentEquation.firstNumber = currentEquation.firstNumber * 10 + number;
    }
  }

  function toggleAddingDecimal() {
    if (
      !canAddNumber ||
      currentEquation.firstNumber
        .toString()
        .split("")
        .find((value) => value === ".")
    ) {
      return;
    }

    isAddingDecimal = !isAddingDecimal;
  }

  function changeNegative() {
    numberNegative = !numberNegative;
  }

  function clear() {
    if (currentEquation.firstNumber != 0 || currentEquation.secondNumber != 0) {
      if (currentOperation) {
        currentEquation.secondNumber = 0;
      } else {
        currentEquation.firstNumber = 0;
      }
      isAddingDecimal = false;
      numberNegative = false;
      zerosAdded = 0;
      currentOperation = null;

      return;
    }
  }

  // calculator animation
  let showCalculator = $state(false);

  $effect(() => {
    showCalculator = true;
  });
</script>

<div class="calculator-container">
  {#if showCalculator}
    <div
      class="calculator"
      in:fly={{ y: "150%", easing: circOut, duration: 400, delay: 100 }}
    >
      <div class="calculator-output">
        {outputValue}
      </div>

      <div class="calculator-grid">
        <div class="number-grid">
          <button on:click={clear}>
            {currentEquation.firstNumber == 0 ? "AC" : "C"}
          </button>
          <button on:click={changeNegative}>+/-</button>
          <button on:click={toggleAddingDecimal}>.</button>
          {#each new Array(10).fill(null).map((_, i) => 9 - i) as item}
            <button
              class:triple-width={item === 0}
              on:click={() => insertNumber(item)}
            >
              {item}
            </button>
          {/each}
        </div>

        <div class="operations-grid">
          <button
            class:selected={currentOperation === "div"}
            on:click={() => {
              currentOperation = "div";
            }}
          >
            &divide;
          </button>
          <button
            class:selected={currentOperation === "mul"}
            on:click={() => {
              currentOperation = "mul";
            }}
          >
            &times;
          </button>
          <button
            class:selected={currentOperation === "add"}
            on:click={() => {
              currentOperation = "add";
            }}
          >
            +
          </button>
          <button
            class:selected={currentOperation === "sub"}
            on:click={() => {
              currentOperation = "sub";
            }}
          >
            -
          </button>
          <button on:click={() => {}}>=</button>
        </div>
      </div>
    </div>
  {/if}
</div>

<style>
  button {
    padding: 1rem;
    border: none;
    cursor: pointer;
    background-color: rgba(255, 255, 255, 0.4);
    border-radius: 0.5rem;
    box-shadow: 0 0.2rem 0.3rem rgba(0, 0, 0, 0.1);
    transition: 200ms ease-in-out;
    transition-property: transform, background-color;
    font-size: 1.5rem;
  }

  button:hover {
    opacity: 0.8;
    transform: translateY(-2px);
  }

  button:active {
    opacity: 0.6;
  }

  .calculator-container {
    display: flex;
    align-items: center;
    justify-content: center;
    height: 100vh;
  }

  .calculator {
    background: rgba(255, 255, 255, 0.5);
    border-radius: 1rem;
    box-shadow: 0 0.2rem 1rem rgba(0, 0, 0, 0.4);
    padding: 1rem;
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  .calculator-output {
    padding: 1rem;
    text-align: right;
    background: rgba(255, 255, 255, 0.62);
    border-radius: 0.5rem;
    box-shadow: 0 4px 30px rgba(0, 0, 0, 0.1);
    backdrop-filter: blur(10px);
    -webkit-backdrop-filter: blur(10px);
    font-size: 2rem;
  }

  .calculator-grid {
    display: grid;
    gap: 1rem;
    grid-template-columns: repeat(4, 1fr);
  }

  .number-grid {
    display: grid;
    gap: 1rem;
    grid-template-columns: repeat(3, 1fr);
    grid-column: span 3;
  }

  .operations-grid {
    display: grid;
    gap: 1rem;
    grid-template-columns: repeat(1, 1fr);
  }

  .operations-grid button {
    background-color: rgb(255, 157, 0, 0.7);
    color: white;
  }

  .operations-grid .selected {
    background-color: rgba(175, 108, 0, 0.7);
  }

  .triple-width {
    grid-column: span 3;
  }
</style>
