<template>
  <div>
    <table class="lectures-table">
      <tr>
        <th>학수번호</th>
        <th>강의명</th>
        <th>교수</th>
        <th>학점</th>
        <th>시간</th>
        <th>강의실</th>
      </tr>
      <tr
        v-for="lecture in lectures"
        @mouseenter="$emit('lecture-mouseenter', lecture)"
        @mouseleave="$emit('lecture-mouseenter', null)"
        @click="$emit('lecture-click', lecture)"
        :class="{ overlapped: isOverlap(lecture, filledTimes) }"
      >
        <td class="nowrap short">{{ lecture.identifier }}</td>
        <td>{{ lecture.title }}</td>
        <td class="short">{{ lecture.professor }}</td>
        <td class="short">{{ lecture.credit }}</td>
        <td class="short">{{ displayTime(lecture.times) }}</td>
        <td class="short">{{ displayRoom(lecture.times) }}</td>
      </tr>
    </table>
  </div>
</template>

<script>
  import { shortWeekdays } from '@/util/constants';
  import { isOverlap } from '@/util';

  export default {
    name: 'lecture-list',
    props: {
      lectures: {
        type: Array,
        default() {
          return [];
        },
      },
      'filled-times': {
        type: Array,
        default() {
          return [];
        },
      },
    },
    methods: {
      isOverlap(lecture, filledTimes) {
        return isOverlap(lecture, filledTimes);
      },
      displayTime(times) {
        if (!times) return '';

        const formattedTimes = times.map((time) => {
          const weekday = shortWeekdays[time.weekday];
          if (time.periodBegin && time.periodEnd) {
            return time.periodBegin === time.periodEnd
              ? `${weekday}${time.periodBegin}`
              : `${weekday}${time.periodBegin}-${time.periodEnd}`;
          }

          const pad = number => (`${number}`.length === 1 ? `0${number}` : `${number}`);

          const beginHour = pad(Math.floor(time.timeBegin / 60));
          const beginMinute = pad(time.timeBegin % 60);
          const endHour = pad(Math.floor(time.timeEnd / 60));
          const endMinute = pad(time.timeEnd % 60);
          return `${weekday}${beginHour}:${beginMinute}-${endHour}:${endMinute}`;
        });
        return formattedTimes.join(', ');
      },
      displayRoom(times) {
        if (!times) return '';

        return Array.from(new Set(times.map(time => time.room))).join(', ');
      },
    },
  };
</script>

<style lang="scss">
  .lectures-table {
    border-collapse: collapse;
    width: 100%;

    th {
      background-color: #f0f0f0;
    }

    th {
      word-break: keep-all;
    }
    th, td {
      border-bottom: 1px solid #e4e4e4;
      padding: 4px;
    }

    td {
      cursor: pointer;

      &.nowrap {
        white-space: nowrap;
      }
      &.short {
        max-width: 300px;
      }
    }

    tr {
      &.overlapped {
        color: #ccc;
      }

      &:nth-child(odd) {
        background-color: #f8f8f8;
      }

      &:hover {
        background-color: #ffffcc;
      }
    }
  }
</style>
