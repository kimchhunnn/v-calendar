<template>
  <div
    class="vc-time-picker"
    :class="[{ 'vc-disabled': isDisabled, 'vc-bordered': showBorder }]"
  >
    <div>
      <svg
        fill="none"
        stroke-linecap="round"
        stroke-linejoin="round"
        stroke-width="2"
        viewBox="0 0 24 24"
        class="vc-time-icon"
        stroke="currentColor"
      >
        <path d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" />
      </svg>
    </div>
    <div class="vc-date-time">
      <div v-if="date" class="vc-date">
        <span class="vc-weekday">
          {{ locale.format(date, 'WWW') }}
        </span>
        <span class="vc-month">
          {{ locale.format(date, 'MMM') }}
        </span>
        <span class="vc-day">
          {{ locale.format(date, 'D') }}
        </span>
        <span class="vc-year">
          {{ locale.format(date, 'YYYY') }}
        </span>
      </div>
      <div class="vc-time">
        <time-select v-model.number="hours" :options="hourOptions" />
        <span style="margin: 0 4px;">:</span>
        <time-select v-model.number="minutes" :options="minuteOptions" />
        <div
          v-if="!is24hr"
          class="vc-am-pm"
          :class="{ 'vc-disabled': !(hours >= 0) }"
        >
          <button
            :class="{ active: isAM }"
            @click.prevent="isAM = true"
            type="button"
          >
            AM
          </button>
          <button
            :class="{ active: !isAM }"
            @click.prevent="isAM = false"
            type="button"
          >
            PM
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import TimeSelect from './TimeSelect';
import { pad } from '../utils/helpers';

export default {
  name: 'TimePicker',
  components: { TimeSelect },
  props: {
    value: { type: Object, required: true },
    locale: { type: Object, required: true },
    theme: { type: Object, required: true },
    is24hr: { type: Boolean, default: true },
    minuteIncrement: { type: Number, default: 1 },
    showBorder: Boolean,
    isDisabled: Boolean,
  },
  data() {
    return {
      hours: '',
      minutes: '',
      isAM: true,
    };
  },
  computed: {
    date() {
      let date = this.locale.normalizeDate(this.value);
      if (this.value.hours === 24) {
        date = new Date(date.getTime() - 1);
      }
      return date;
    },
    hourOptions() {
      const options12 = [
        { value: 0, label: '12' },
        { value: 1, label: '1' },
        { value: 2, label: '2' },
        { value: 3, label: '3' },
        { value: 4, label: '4' },
        { value: 5, label: '5' },
        { value: 6, label: '6' },
        { value: 7, label: '7' },
        { value: 8, label: '8' },
        { value: 9, label: '9' },
        { value: 10, label: '10' },
        { value: 11, label: '11' },
      ];
      const options24 = [
        { value: 0, label: '00' },
        { value: 1, label: '01' },
        { value: 2, label: '02' },
        { value: 3, label: '03' },
        { value: 4, label: '04' },
        { value: 5, label: '05' },
        { value: 6, label: '06' },
        { value: 7, label: '07' },
        { value: 8, label: '08' },
        { value: 9, label: '09' },
        { value: 10, label: '10' },
        { value: 11, label: '11' },
        { value: 12, label: '12' },
        { value: 13, label: '13' },
        { value: 14, label: '14' },
        { value: 15, label: '15' },
        { value: 16, label: '16' },
        { value: 17, label: '17' },
        { value: 18, label: '18' },
        { value: 19, label: '19' },
        { value: 20, label: '20' },
        { value: 21, label: '21' },
        { value: 22, label: '22' },
        { value: 23, label: '23' },
      ];

      if (this.is24hr) return options24;
      return options12;
    },
    minuteOptions() {
      const options = [];
      let m = 0;
      let added = false;
      while (m <= 59) {
        options.push({
          value: m,
          label: pad(m, 2),
        });
        added = added || m === this.minutes;
        m += this.minuteIncrement;
        // Add disabled option if interval has skipped it
        if (!added && m > this.minutes) {
          added = true;
          options.push({
            value: this.minutes,
            label: pad(this.minutes, 2),
            disabled: true,
          });
        }
      }
      return options;
    },
  },
  watch: {
    value() {
      this.setup();
    },
    hours() {
      this.updateValue();
    },
    minutes() {
      this.updateValue();
    },
    isAM() {
      this.updateValue();
    },
  },
  created() {
    this.setup();
  },
  methods: {
    protected(fn) {
      if (this.busy) return;
      this.busy = true;
      fn();
      this.$nextTick(() => (this.busy = false));
    },
    setup() {
      this.protected(() => {
        let { hours } = this.value;
        if (hours === 24) hours = 0;
        let isAM = true;
        if (!this.is24hr && hours >= 12) {
          hours -= 12;
          isAM = false;
        }
        this.hours = hours;
        this.minutes = this.value.minutes;
        this.isAM = isAM;
      });
    },
    updateValue() {
      this.protected(() => {
        let hours = this.hours;
        if (!this.is24hr && !this.isAM) {
          hours += 12;
        }
        this.$emit('input', {
          ...this.value,
          hours,
          minutes: this.minutes,
          seconds: 0,
          milliseconds: 0,
        });
      });
    },
  },
};
</script>

<style lang="postcss" scoped>
.vc-time-picker {
  display: flex;
  align-items: center;
  padding: 8px;
  &.vc-invalid {
    pointer-events: none;
    opacity: 0.5;
  }
  &.vc-bordered {
    border-top: 1px solid var(--gray-400);
  }
}

.vc-date-time {
  margin-left: 8px;
}

.vc-disabled {
  pointer-events: none;
  opacity: 0.5;
}

.vc-time-icon {
  width: 16px;
  height: 16px;
  color: var(--gray-600);
}

.vc-date {
  display: flex;
  align-items: center;
  font-size: var(--text-sm);
  font-weight: var(--font-semibold);
  text-transform: uppercase;
  padding: 0 0 4px 4px;
  margin-top: -4px;
  & .vc-weekday {
    color: var(--gray-700);
    letter-spacing: var(--tracking-wide);
  }
  & .vc-month {
    color: var(--accent-600);
    margin-left: 8px;
  }
  & .vc-day {
    color: var(--accent-600);
    margin-left: 4px;
  }
  & .vc-year {
    color: var(--gray-500);
    margin-left: 8px;
  }
}

.vc-time {
  display: flex;
  align-items: center;
}

.vc-am-pm {
  display: flex;
  align-items: center;
  background: var(--gray-200);
  margin-left: 8px;
  padding: 4px;
  border-radius: var(--rounded);
  height: 30px;
  & button {
    color: var(--gray-900);
    font-size: var(--text-sm);
    font-weight: var(--font-medium);
    padding: 0 4px;
    background: transparent;
    border: 2px solid transparent;
    border-radius: var(--rounded);
    line-height: var(--leading-snug);
    &:hover {
      color: var(--gray-600);
    }
    &:focus {
      border-color: var(--accent-400);
    }
    &.active {
      background: var(--accent-600);
      color: var(--white);
      &:hover {
        background: var(--accent-500);
      }
      &:focus {
        border-color: var(--accent-400);
      }
    }
  }
}

.vc-is-dark {
  & .vc-time-picker {
    border-color: var(--gray-700);
  }
  & .vc-time-icon {
    color: var(--gray-400);
  }
  & .vc-weekday {
    color: var(--gray-400);
  }
  & .vc-month {
    color: var(--accent-400);
  }
  & .vc-day {
    color: var(--accent-400);
  }
  & .vc-year {
    color: var(--gray-500);
  }
  & .vc-am-pm {
    background: var(--gray-700);
    &:focus {
      border-color: var(--accent-500);
    }
    & button {
      color: var(--gray-100);
      &:hover {
        color: var(--gray-400);
      }
      &:focus {
        border-color: var(--accent-500);
      }
      &.active {
        background: var(--accent-500);
        color: var(--white);
        &:hover {
          background: var(--accent-600);
        }
        &:focus {
          border-color: var(--accent-500);
        }
      }
    }
  }
}
</style>
