<template>
  <div>
    <CalendarDay v-if="modal" />
    <section v-else class="section" style="font-family: 'Single Day', cursive;">
      <div class="container">
        <h2 class="subtitle has-text-centered">
          <button class="button is-small is-primary is-outlined mr-5"
          @click="calendarData(-1)">&lt;</button>
          {{ year }}년 {{ month }}월
          <button class="button is-small is-primary is-outlined ml-5"
          @click="calendarData(1)">&gt;</button>
        </h2>
        <table class="table has-text-centered is-fullwidth">
          <thead>
            <th v-for="day in days" :key="day">{{ day }}</th>
          </thead>
          <tbody>
            <tr class="date"  v-for="(date, idx) in dates" :key="idx">
              <td
                class="cursor"
                @click="viewModal(day)"
                v-for="(day, secondIdx) in date"
                :key="secondIdx"
                :class="{ 
                  'has-text-info-dark': idx === 0 && day >= lastMonthStart,
                'has-text-danger': dates.length - 1 === idx && nextMonthStart > day,
                'has-text-primary cursor': day === today && month === currentMonth && year === currentYear
                }"
              >
                {{ day }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <!-- 모달 구간 -->
    <!-- <Modal v-bind:modalOpen="modalOpen" v-on:closeModal="closeModal" /> -->
    </section>
  </div>
</template>

<script>
import CalendarDay from "@/components/CalendarDay.vue"

export default {
  components: {
    CalendarDay
  },
  data() {
    return {
      modal: false,
      days: [
        '일요일',
        '월요일',
        '화요일',
        '수요일',
        '목요일',
        '금요일',
        '토요일',
      ],
      selectDay :0 ,
      dates: [],
      currentYear: 0,
      currentMonth: 0,
      year: 0,
      month: 0,
      lastMonthStart: 0,
      nextMonthStart: 0,
      today: 0,
      modalOpen:false,
    };

  
  },
  created() { // 데이터에 접근이 가능한 첫 번째 라이프 사이클
    const date = new Date();
    this.currentYear = date.getFullYear(); // 이하 현재 년, 월 가지고 있기
    this.currentMonth = date.getMonth() + 1;
    this.year = this.currentYear;
    this.month = this.currentMonth;
    this.today = date.getDate(); // 오늘 날짜
    this.calendarData();
  },
  methods: {
    viewModal (event) {
      this.selectDay = event
      this.$router.push({
        name: "CalendarDay",
        query: { title: String(this.year), body: this.month, foot: this.selectDay},
      })
      // this.modal = !this.modal
    },
    calendarData(arg) { // 인자를 추가
      if (arg < 0) { // -1이 들어오면 지난 달 달력으로 이동
        this.month -= 1;
      } else if (arg === 1) { // 1이 들어오면 다음 달 달력으로 이동
        this.month += 1;
      }
      if (this.month === 0) { // 작년 12월
        this.year -= 1;
        this.month = 12;
      } else if (this.month > 12) { // 내년 1월
        this.year += 1;
        this.month = 1;
      }
      const [
        monthFirstDay,
        monthLastDate,
        lastMonthLastDate,
      ] = this.getFirstDayLastDate(this.year, this.month);
      this.dates = this.getMonthOfDays(
        monthFirstDay,
        monthLastDate,
        lastMonthLastDate,
      );
    },
    getFirstDayLastDate(year, month) {
      const firstDay = new Date(year, month - 1, 1).getDay(); // 이번 달 시작 요일
      const lastDate = new Date(year, month, 0).getDate(); // 이번 달 마지막 날짜
      let lastYear = year;
      let lastMonth = month - 1;
      if (month === 1) {
        lastMonth = 12;
        lastYear -= 1;
      }
      const prevLastDate = new Date(lastYear, lastMonth, 0).getDate(); // 지난 달 마지막 날짜
      return [firstDay, lastDate, prevLastDate];
    },
    getMonthOfDays(
      monthFirstDay,
      monthLastDate,
      prevMonthLastDate,
    ) {
      let day = 1;
      let prevDay = (prevMonthLastDate - monthFirstDay) + 1;
      const dates = [];
      let weekOfDays = [];
      while (day <= monthLastDate) {
        if (day === 1) {
          // 1일이 어느 요일인지에 따라 테이블에 그리기 위한 지난 셀의 날짜들을 구할 필요가 있다.
          for (let j = 0; j < monthFirstDay; j += 1) {
            if (j === 0) this.lastMonthStart = prevDay; // 지난 달에서 제일 작은 날짜
            weekOfDays.push(prevDay);
            prevDay += 1;
          }
        }
        weekOfDays.push(day);
        if (weekOfDays.length === 7) {
          // 일주일 채우면
          dates.push(weekOfDays);
          weekOfDays = []; // 초기화
        }
        day += 1;
      }
      const len = weekOfDays.length;
      if (len > 0 && len < 7) {
        for (let k = 1; k <= 7 - len; k += 1) {
          weekOfDays.push(k);
        }
      }
      if (weekOfDays.length > 0) dates.push(weekOfDays); // 남은 날짜 추가
      this.nextMonthStart = weekOfDays[0]; // 이번 달 마지막 주에서 제일 작은 날짜
      return dates;
    },
    showModal() {
      this.modalOpen = true;
    },
    closeModal(event) {
      this.modalOpen = event;
    },
  },
};
</script>
<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Single+Day&display=swap');
.cursor:not(.has-text-danger, .has-text-info-dark):hover {
  cursor: pointer;
  color:rgb(174, 213, 121);
}
.has-text-info-dark {
  color:rgb(176, 173, 173);
}
.has-text-primary {
  font-size: 1.1em;
  font-weight: bold;
  color:rgb(108, 181, 161);
}
.has-text-danger{
  color:rgb(176, 173, 173);
}
.section {
  position: absolute;
  top: 20%;
  left: 20%;
  display: flex;
  width: 70%;
}
.date {
  font-size: 1.5em;
}
.subtitle{
  display: flex;
  justify-content: center;
}
</style>