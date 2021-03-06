<template>
  <div class="timetable-container">
    <div class="section-timetable">
      <timetable
        class="timetable"
        v-if="timetable"
        :timetable="timetable"
        :preview-lectures="previewLectures"
        :editable="editable"
        @delete-lecture="(lecture) => deleteLecture(lecture)"
      ></timetable>
    </div>
    <div class="section-search" v-if="editable">
      <search-lecture
        :lectures="lectures"
        :filled-times="filledTimes"
        @lecture-mouseenter="(lecture) => previewAtTimetable(lecture)"
        @lecture-select="(lectureId) => showLectureModal(lectureId)"
      ></search-lecture>
    </div>

    <popup-modal :show="showModal" @dismiss="showModal = false" v-if="modalLecture" class="lecture-modal">
      <lecture-detail :lecture="modalLecture" class="lecture-modal-content"></lecture-detail>
      <button class="btn-add" @click="addLecture(modalLecture)" :disabled="isOverlap(modalLecture, filledTimes)">시간표에 추가</button>
    </popup-modal>
  </div>
</template>

<script>
  import { mapState } from 'vuex';

  import { isOverlap } from '@/util';

  import Timetable from '@/components/Timetable';
  import SearchLecture from '@/components/SearchLecture';
  import LectureDetail from '@/components/LectureDetail';
  import PopupModal from '@/components/PopupModal';

  export default {
    name: 'user-timetable-container',
    components: { Timetable, SearchLecture, LectureDetail, PopupModal },
    data() {
      return {
        previewLectures: [],
        periods: [
          { period: 1, timeBegin: 540, timeEnd: 615 },
          { period: 2, timeBegin: 630, timeEnd: 705 },
          { period: 3, timeBegin: 720, timeEnd: 770 },
          { period: 4, timeBegin: 780, timeEnd: 830 },
          { period: 5, timeBegin: 840, timeEnd: 915 },
          { period: 6, timeBegin: 930, timeEnd: 1005 },
          { period: 7, timeBegin: 1020, timeEnd: 1070 },
          { period: 8, timeBegin: 1080, timeEnd: 1130 },
          { period: 9, timeBegin: 1140, timeEnd: 1190 },
        ],
        showModal: false,
        modalLecture: null,
      };
    },
    computed: {
      editable() {
        if (this.$store.state.me === null || !this.timetable) return false;
        return this.timetable.user.id === this.$store.state.me.id;
      },
      username() {
        return this.$route.params.username;
      },
      user() {
        const userId = this.$store.state.usernames[this.username];
        return this.$store.state.users[userId];
      },
      timetableId() {
        return this.$route.params.timetable_id;
      },
      filledTimes() {
        if (!this.timetable) return [];

        return this.timetable.lectures.reduce((res, lecture) => [
          ...res,
          ...lecture.times,
        ], []);
      },
      semester() {
        if (!this.timetable) return null;

        return this.$store.getters.semesters[this.timetable.semester.id];
      },
      lectures() {
        if (!this.semester || !this.semester.lectures) return [];

        return this.semester.lectures;
      },
      ...mapState({
        timetable(state) {
          return state.timetables[this.timetableId];
        },
      }),
    },
    methods: {
      isOverlap(lecture, filledTimes) {
        return isOverlap(lecture, filledTimes);
      },
      showLectureModal(lecture) {
        this.modalLecture = lecture;
        this.showModal = true;
      },
      previewAtTimetable(lecture) {
        if (lecture === null) {
          this.previewLectures = [];
          return;
        }
        this.previewLectures.push(lecture);
      },
      addLecture(lecture) {
        this.$store.dispatch('ADD_LECTURE_TO_TIMETABLE', { timetableId: this.timetableId, lecture });
      },
      deleteLecture(lectureId) {
        this.$store.dispatch('DELETE_LECTURE_FROM_TIMETABLE', { timetableId: this.timetableId, lectureId });
      },
      refreshTimetable() {
        // TODO: FETCH_USER_TIMETABLE (not all timetableS)
        this.$store.dispatch('FETCH_USER_TIMETABLES', { user_id: this.user.id });
      },
      refreshLectures() {
        this.$store.dispatch('FETCH_SEMESTER_LECTURES', { semesterId: this.semester.id });
      },
    },
    watch: {
      timetableId() {
        this.refreshTimetable();
      },
      semester() {
        this.refreshLectures();
      },
    },
    created() {
      if (this.semester) {
        this.refreshLectures();
      }
    },
  };
</script>

<style lang="scss">
  .timetable-container {
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    padding: 4px;

    .section-timetable, .section-search {
      padding: 4px;
      overflow-y: scroll;
    }

    .section-timetable {
      max-width: 960px;
      margin: 0 auto;
    }

    .timetable {
      width: 100%;
      box-sizing: border-box;
    }

    .section-timetable {
      flex: 1;

      min-width: 500px;

      .timetable {
        height: 100%;
        min-height: 600px;
      }
    }

    .section-search {
      flex: 1.5;
    }

    .lecture-modal {
      .lecture-modal-content {
        padding: 16px 8px;
      }

      button {
        width: 100%;
      }

      .btn-add {
        background-color: #5555ff;
        color: white;
        border: 0;
        font-size: 12pt;
        font-weight: bold;
        padding: 16px;

        &:hover {
          background-color: #4f4fec;
        }

        &:disabled {
          background-color: #ddd;
        }
      }
    }
  }
</style>
