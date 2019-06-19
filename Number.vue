<template>
  <div class="number">
    <b-input-group>
      <!-- Prepend -->
      <b-input-group-text
        v-if="['currency'].includes(type)"
        slot="prepend">
        <font-awesome-icon
          tabindex="-1"
          @click="$refs.number.focus()"
          :icon="iconMap[type]">
        </font-awesome-icon>
      </b-input-group-text>
      <!-- / Prepend -->

      <!-- Amount -->
      <input
        class="form-control"
        type="text"
        :id="id"
        :name="name"
        :placeholder="placeholder"
        :readonly="readonly"
        :disabled="disabled"
        :size="size"
        :value="displayAmount"
        @keydown="handleKeyPress"
        @wheel="stepAmount && wheel($event)"
        @blur="update($refs.number.value)"
        ref="number"
        v-select-on-focus
      />
      <!-- / Amount -->

      <!-- Append -->
      <b-input-group-text
        class="p-0"
        slot="append">
        <!-- Append Icon -->
        <div
          v-if="['percent'].includes(type)"
          class="append-item">
          <font-awesome-icon
            tabindex="-1"
            @click="$refs.number.focus()"
            :icon="iconMap[type]">
          </font-awesome-icon>
        </div>
        <!-- / Append Icon -->

        <!-- General Append -->
        <div
          v-if="appendText"
          class="append-item"
          v-html="appendText">
        </div>
        <!-- / General Append -->

        <!-- Stepper -->
        <div
          v-if="stepAmount"
          class="stepper d-flex flex-column">
          <button
            type="button"
            tabindex="-1"
            @click="stepChange(stepAmount)">
            <font-awesome-icon
              icon="caret-up">
            </font-awesome-icon>
          </button>
          <button
            type="button"
            tabindex="-1"
            @click="stepChange(-stepAmount)">
            <font-awesome-icon
              icon="caret-down">
            </font-awesome-icon>
          </button>
        </div>
        <!-- / Stepper -->
      </b-input-group-text>
      <!-- / Append -->
    </b-input-group>
  </div>
</template>

<script>
import Formatters from '../services/formatters';

export default {
  name: 'number',
  props: [
    'value',
    'id',
    'name',
    'type',
    'placeholder',
    'step',
    'displayDecimals',
    'valueDecimals',
    'min',
    'max',
    'readonly',
    'disabled',
    'size',
    'appendText',
  ],
  data() {
    const isPercent = this.type === 'percent';
    const divisor = isPercent ? 100 : 1;
    let maxAmount = isPercent ? 100 : Infinity;
    let displayDecimals = 2;
    let valueDecimals = isPercent ? 4 : 2;

    if (this.max) {
      maxAmount = parseFloat(this.max);
    }

    if (this.displayDecimals) {
      displayDecimals = parseInt(this.displayDecimals);
    }

    if (this.valueDecimals) {
      valueDecimals = parseInt(this.valueDecimals);
    }

    return {
      maxAmount,
      divisor,
      dispDec: displayDecimals,
      valDec: valueDecimals,
      amount: '',
      stepAmount: this.step ? parseFloat(this.step) : 0,
      minAmount: this.min ? parseFloat(this.min) : 0,
      iconMap: {
        currency: 'dollar-sign',
        number: 'hashtag',
        percent: 'percentage',
      },
    };
  },
  computed: {
    displayAmount() {
      const amount = parseFloat((this.amount * this.divisor).toFixed(this.dispDec));
      return isNaN(this.amount) ? '' : Formatters.number(amount, this.dispDec);
    },
  },
  watch: {
    amount(value) {
      this.$emit('input', value);
    },
    value(value) {
      this.amount = value;
    },
    max(value) {
      this.maxAmount = parseFloat(value);
    },
    min(value) {
      this.minAmount = parseFloat(value);
    },
  },
  methods: {
    handleKeyPress(e) {
      // eslint-disable-next-line
      const allowedChars = /\d|\,|\.|\-|\Backspace|\Tab|\Enter/;
      const keyCode = e.keyCode || e.which;
      const match = e.key.match(allowedChars);

      // if ctrl or cmd + A (select all) pressed, select all text
      if ((e.metaKey || e.ctrlKey) && keyCode === 65) {
        this.$refs.number.select();
      // if enter pressed
      } else if (keyCode === 13) {
        this.update(this.$refs.number.value);
      // if up arrow pressed
      } else if (keyCode === 38) {
        this.stepChange(this.stepAmount);
      // if down arrow pressed
      } else if (keyCode === 40) {
        this.stepChange(-this.stepAmount);
      }

      if (!match) {
        e.preventDefault();
      }

      return !!match;
    },
    update(a) {
      let parsed = parseFloat(a.replace(/,/g, ''));

      // validate min/max
      if (parsed > this.maxAmount) {
        parsed = this.maxAmount;
      } else if (parsed < this.minAmount) {
        parsed = this.minAmount;
      }

      this.amount = parseFloat((parsed / this.divisor).toFixed(this.valDec));
      this.$forceUpdate();
    },
    stepChange(diff) {
      let displayAmount = '0.00';

      if (this.displayAmount) {
        displayAmount = this.displayAmount.replace(/,/g, '');
      }

      const amount = (parseFloat(displayAmount) + diff).toFixed(this.dispDec);
      this.update(amount);
    },
    wheel(e) {
      const detectMouseWheelDirection = () => {
        let delta = null;
        let direction = false;

        if (!e) { // if the event is not provided, we get it from the window object
          e = window.event;
        }

        if (e.wheelDelta) { // will work in most cases
          delta = e.wheelDelta / 60;
        } else if (e.detail) { // fallback for Firefox
          delta = -e.detail / 2;
        }

        if (delta !== null) {
          direction = delta > 0 ? 1 : -1;
        }

        return direction;
      };

      const dir = detectMouseWheelDirection();
      this.stepChange(dir * this.stepAmount);
    },
  },
  created() {
    this.amount = this.value;
  },
};
</script>

<style lang="scss">
.number {
  .input-group-append {
    .input-group-text:empty {
      display: none;
    }
  }

  .append-item {
    padding: 0.375rem 0.75rem;
    border-right: 1px solid #ced4da;


    &:last-child {
      border-right: none;
    }
  }

  .stepper {
    height: 100%;
    button {
      border: none;
      background: none;
      padding: 0;
      outline: none;
      overflow: hidden;
      display: block;
      line-height: 13px;
      padding: 0 .5rem;

      svg {
        font-size: 13px;
        color: #495057;
      }

      &:first-child {
        height: calc(50% + 1px);
        border-bottom: 1px solid #ced4da;
      }

      &:last-child {
        height: calc(50% - 1px);
      }
    }
  }
}
</style>
