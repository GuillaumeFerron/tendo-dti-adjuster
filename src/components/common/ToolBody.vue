<template>
  <div class="card mb-0 w-75">
    <div class="card-body px-5">
      <div class="d-flex justify-content-between align-items-center">
        <div>
          <h3>Tendo DtI Adjuster</h3>
        </div>
        <div v-if="tooltipContent.length > 0" v-tooltip="{
          content: tooltipContent,
          html: true,
          hideTriggers: ['click'],
          autoHide: true
        }" class="clickable">
          <h3>ⓘ</h3>
        </div>
      </div>
      <DtiAdjuster/>
    </div>
  </div>
</template>

<script>
import DtiAdjuster from '../DtiAdjuster.vue'

export default {
  components: {
    DtiAdjuster
  },
  data() {
    return {
      tooltip: {
        old: 'https://docs.google.com/spreadsheets/d/1DvqssBXmK2ubR652HEP5Qr1d7gC31Dy4ucT7-NO8XE4/edit?gid=0#gid=0',
        guide: 'https://scribehow.com/shared/Allowing_users_to_use_their_remaining_CL__xNxvEkIIR4uKg0bxsHNT0A'
      }
    }
  },
  computed: {
    tooltipContent() {
      return (this.tooltip.guide ? `Guide: <a href="${this.tooltip.guide}" target="_blank">Here</a><br>` : '') + (this.tooltip.old ? `Legacy: <a href="${this.tooltip.old}" target="_blank">Here</a>` : '')
    }
  }
}
</script>

<style lang="scss" scoped>
.card {
  -webkit-box-shadow: 5px 5px 15px 5px rgba(0, 0, 0, 0.3);
  box-shadow: 5px 5px 15px 5px rgba(0, 0, 0, 0.3);
}
</style>
