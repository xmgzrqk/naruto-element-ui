<template>
  <table @click="handleQuarterTableClick" @mousemove="handleMouseMove" class="el-quarter-table">
    <tbody>
    <tr v-for="(row, key) in rows" :key="key">
      <td :class="getCellStyle(cell)" v-for="(cell, key) in row" :key="key">
        <div>
          <a class="cell">{{ t('el.datepicker.quarter' + (cell.text + 1) ) }}</a>
        </div>
      </td>
    </tr>
    </tbody>
  </table>
</template>

<script type="text/babel">
  import Locale from 'element-ui/src/mixins/locale';
  import { isDate, range, getDayCountOfMonth, nextDate, getMonthNumberInQuarter } from 'element-ui/src/utils/date-util';
  import { hasClass } from 'element-ui/src/utils/dom';
  import { arrayFindIndex, coerceTruthyValueToArray, arrayFind } from 'element-ui/src/utils/util';

  const datesInQuarter = (year, quarter) => {
    const monthList = getMonthNumberInQuarter(quarter);
    const numOfDays = monthList
      .map(m => getDayCountOfMonth(year, m))
      .reduce((prev, cur) => prev + cur, 0);
    const firstDay = new Date(year, monthList[0], 1);
    return range(numOfDays).map(n => nextDate(firstDay, n));
  };

  const getMonthTimestamp = function(time) {
    if (typeof time === 'number' || typeof time === 'string') {
      return new Date(time).getTime();
    } else if (time instanceof Date) {
      return time.getTime();
    } else {
      return NaN;
    }
  };

  export default {
    props: {
      disabledDate: {},
      value: {},
      selectionMode: {
        default: 'quarter'
      },
      defaultValue: {
        validator(val) {
          // null or valid Date Object
          return val === null || (val instanceof Date && isDate(val)) || (Array.isArray(val) && val.every(isDate));
        }
      },
      date: {},
      minDate: {},
      maxDate: {},
      rangeState: {
        default() {
          return {
            endDate: null,
            selecting: false
          };
        }
      }
    },
    mixins: [Locale],

    watch: {
      'rangeState.endDate'(newVal) {
        this.markRange(this.minDate, newVal);
      },

      minDate(newVal, oldVal) {
        if (getMonthTimestamp(newVal) !== getMonthTimestamp(oldVal)) {
          this.markRange(this.minDate, this.maxDate);
        }
      },

      maxDate(newVal, oldVal) {
        if (getMonthTimestamp(newVal) !== getMonthTimestamp(oldVal)) {
          this.markRange(this.minDate, this.maxDate);
        }
      }
    },

    data() {
      return {
        tableRows: [ [] ],
        lastRow: null,
        lastColumn: null
      };
    },

    methods: {
      getCellStyle(cell) {
        const style = {};
        const year = this.date.getFullYear();
        const today = new Date();
        const quarter = cell.text + 1;
        style.disabled = typeof this.disabledDate === 'function'
          ? datesInQuarter(year, quarter).some(this.disabledDate)
          : false;
        const monthList = getMonthNumberInQuarter(quarter);
        const monthMatch = month => monthList[0] === month ||
          monthList[1] === month ||
          monthList[2] === month;
        style.current = arrayFindIndex(coerceTruthyValueToArray(this.value), date => date.getFullYear() === year) >= 0 &&
          arrayFindIndex(coerceTruthyValueToArray(this.value), date => monthMatch(date.getMonth())) >= 0;
        style.today = today.getFullYear() === year && monthMatch(today.getMonth());
        style.default = this.defaultValue &&
          this.defaultValue.getFullYear() === year &&
          monthMatch(this.defaultValue.getMonth());

        if (cell.inRange) {
          style['in-range'] = true;

          if (cell.start) {
            style['start-date'] = true;
          }

          if (cell.end) {
            style['end-date'] = true;
          }
        }
        return style;
      },

      getMonthOfCell(monthList) {
        const len = monthList.length;
        const year = this.date.getFullYear();
        return [new Date(year, monthList[0], 1), new Date(year, monthList[len - 1] + 1, 0)];
      },

      getQuarterStartDate(date) {
        const quarter = Math.floor(date.getMonth() / 3);
        const year = date.getFullYear();
        const month = quarter * 3;
        return new Date(year, month, 1);
      },

      getQuarterEndDate(date) {
        const quarter = Math.floor(date.getMonth() / 3) + 1;
        const year = date.getFullYear();
        const month = quarter * 3;
        return new Date(year, month, 0);
      },

      markRange(minDate, maxDate) {
        if (!minDate && !maxDate) return;
        const endDate = this.getQuarterEndDate(minDate);
        minDate = getMonthTimestamp(minDate);
        maxDate = getMonthTimestamp(maxDate) || endDate;
        const minDateVal = getMonthTimestamp(this.getQuarterStartDate(new Date(Math.min(minDate, maxDate))));
        const maxDateVal = getMonthTimestamp(this.getQuarterEndDate(new Date(Math.max(minDate, maxDate))));
        [minDate, maxDate] = [minDateVal, maxDateVal];
        const rows = this.rows;
        for (let i = 0, k = rows.length; i < k; i++) {
          const row = rows[i];
          for (let j = 0, l = row.length; j < l; j++) {

            const cell = row[j];
            const quarter = i * 4 + j + 1;
            const monthList = getMonthNumberInQuarter(quarter);
            const dateRange = this.getMonthOfCell(monthList);
            cell.inRange = minDate && dateRange[0].getTime() >= minDate && dateRange[1].getTime() <= maxDate;
            cell.start = minDate && dateRange[0].getTime() === minDate;
            cell.end = maxDate && dateRange[1].getTime() === maxDate;
          }
        }
      },

      handleMouseMove(event) {
        if (!this.rangeState.selecting) return;

        let target = event.target;
        if (target.tagName === 'A') {
          target = target.parentNode.parentNode;
        }
        if (target.tagName === 'DIV') {
          target = target.parentNode;
        }
        if (target.tagName !== 'TD') return;

        const row = target.parentNode.rowIndex;
        const column = target.cellIndex;
        // can not select disabled date
        if (this.rows[row][column].disabled) return;

        // only update rangeState when mouse moves to a new cell
        // this avoids frequent Date object creation and improves performance
        const monthList = getMonthNumberInQuarter(row * 4 + column + 1);
        const dateRange = this.getMonthOfCell(monthList);
        if (row !== this.lastRow || column !== this.lastColumn) {
          this.lastRow = row;
          this.lastColumn = column;
          this.$emit('changerange', {
            minDate: this.minDate,
            maxDate: this.maxDate,
            rangeState: {
              selecting: true,
              endDate: dateRange[1]
            }
          });
        }
      },

      handleQuarterTableClick(event) {
        let target = event.target;
        if (target.tagName === 'A') {
          target = target.parentNode.parentNode;
        }
        if (target.tagName === 'DIV') {
          target = target.parentNode;
        }
        if (target.tagName !== 'TD') return;
        if (hasClass(target, 'disabled')) return;
        const column = target.cellIndex;
        const row = target.parentNode.rowIndex;
        const quarter = row * 4 + column + 1;
        const monthList = getMonthNumberInQuarter(quarter);
        const dateRange = this.getMonthOfCell(monthList);
        if (this.selectionMode === 'range') {
          if (!this.rangeState.selecting) {
            this.$emit('pick', {minDate: dateRange[0], maxDate: null});
            this.rangeState.selecting = true;
          } else {
            if (dateRange[0] >= this.minDate) {
              this.$emit('pick', {minDate: this.minDate, maxDate: dateRange[1]});
            } else {
              this.$emit('pick', {minDate: dateRange[0], maxDate: this.getQuarterEndDate(this.minDate)});
            }
            this.rangeState.selecting = false;
          }
        } else {
          this.$emit('pick', quarter);
        }
      }
    },

    computed: {
      rows() {
        // TODO: refactory rows / getCellClasses
        const rows = this.tableRows;
        const disabledDate = this.disabledDate;
        const selectedDate = [];
        const now = getMonthTimestamp(new Date());

        for (let i = 0; i < 1; i++) {
          const row = rows[i];
          for (let j = 0; j < 4; j++) {
            let cell = row[j];
            if (!cell) {
              cell = { row: i, column: j, type: 'normal', inRange: false, start: false, end: false };
            }

            cell.type = 'normal';

            const index = i * 4 + j;
            const time = new Date(this.date.getFullYear(), index).getTime();
            const monthList = getMonthNumberInQuarter(index + 1);
            const dateRange = this.getMonthOfCell(monthList);
            cell.inRange = dateRange[0].getTime() >= getMonthTimestamp(this.minDate) && dateRange[1].getTime() <= getMonthTimestamp(this.maxDate);
            cell.start = this.minDate && dateRange[0].getTime() === getMonthTimestamp(this.minDate);
            cell.end = this.maxDate && dateRange[1].getTime() === getMonthTimestamp(this.maxDate);
            const isToday = time === now;

            if (isToday) {
              cell.type = 'today';
            }
            cell.text = index;
            let cellDate = new Date(time);
            const year = this.date.getFullYear();
            cell.disabled = typeof disabledDate === 'function'
              ? datesInQuarter(year, index + 1).some(disabledDate)
              : false;
            cell.selected = arrayFind(selectedDate, date => date.getTime() === cellDate.getTime());

            this.$set(row, j, cell);
          }
        }
        return rows;
      }
    }
  };
</script>
