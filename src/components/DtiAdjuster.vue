<template>
  <div class="w-100">
    <div class="row">
      <h5>Calculator</h5>
      <div class="col-12 col-lg-6">
        <div>
          <label for="dti">DtI (%)</label>
          <input type="number" class="form-control" step="1" id="dti" v-model="data.dti">
        </div>
        <div class="mt-3">
          <label for="payroll">Number of Payroll Dates</label>
          <select name="Payroll" id="payroll" class="form-control" v-model="data.payroll">
            <option value="1">1</option>
            <option value="2">2</option>
          </select>
        </div>
        <div class="mt-3">
          <label for="cl">Credit Limit</label>
          <input type="number" class="form-control" step=".01" id="cl" v-model="data.cl">
        </div>
        <div class="mt-3">
          <label for="tenure">Loan Tenure (months)</label>
          <input type="number" class="form-control" step="1" id="tenure" v-model="data.tenure">
        </div>
      </div>
      <div class="col-12 col-lg-6">
        <div>
          <label for="next-payment">Next Payment&emsp;</label>
          <small><i>{{ data.cl || 0 }} / ({{ data.tenure || 0 }} * 2)
              / {{
              data.payroll || 2
              }}</i></small>
          <input type="number" class="form-control" disabled id="next-payment" :value="nextPayment">
        </div>
        <div class="mt-3">
          <label for="net-income">Net Income&emsp;</label>
          <small><i>({{ nextPayment || 0 }} * {{ data.dti || 0 }}%)
              * {{
              data.tenure || 0
              }}</i></small>
          <input type="number" class="form-control" disabled id="net-income" :value="netIncome">
        </div>
        <div class="mt-3">
          <p class="py-1"></p>
          <button class="form-control btn btn-light" @click="clear">Clear</button>
        </div>
      </div>
    </div>
    <hr class="mt-4">
    <div class="row mt-3">
      <h5>Process</h5>
      <div class="my-2">
        <i>1. Update the user's net income to <b>PHP{{ netIncome }}</b></i>
      </div>
      <div class="my-2">
        <i>2. Add a comment to the users account (make sure to mention the previous net income):</i><br>
        <div class="bg-light rounded p-2 mt-1">
          <code id="copy-area">
            {{ comment }}
          </code>
        </div>
        <div class="w-25">
          <p class="py-1"></p>
          <button class="form-control btn btn-primary" @click="copy">{{ copied ? 'Copied!' : 'Copy' }}</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  const CACHE_KEY = 'TENDOPAY_DTI_ADJUSTER'
  const FORCE_CACHE = true

  export default {
    data() {
      return {
        data: {
          dti: 30,
          payroll: 2,
          cl: null,
          tenure: 2
        },
        cacheProcessed: false,
        copied: false
      }
    },
    watch: {
      data: {
        deep: true,
        handler() {
          if (this.cacheProcessed) {
            this.storeCache()
          }
        }
      }
    },
    mounted() {
      this.restoreCache()
    },
    computed: {
      nextPayment () {
        return (Math.ceil(this.data.cl / (this.data.tenure * 2) / this.data.payroll * 100) / 100) || 0
      },
      netIncome() {
        return Math.ceil((this.nextPayment / (this.data.dti / 100)) * this.data.payroll / 100) * 100 || 0
      },
      comment() {
        return`Updated Net Income from ___ to ${this.netIncome} for the user to avail their whole CL (consideration)`
      }
    },
    methods: {
      storeCache() {
        const obj = {
          data: this.data
        }
        localStorage.setItem(CACHE_KEY, JSON.stringify(obj))
      },
      restoreCache() {
        const res = localStorage.getItem(CACHE_KEY)
        if (res) {
          let clear = {}
          try {
            clear = JSON.parse(res)
          } catch (err) {
            clear = {}
          }
          if (!FORCE_CACHE) {
            this.clear()
            console.log(`* DTI ADJUSTER: Invalidated cache`)
          } else {
            const data = clear.data
            Object.keys(data || {}).forEach(_ => {
              if (data[_] != null) {
                this.data[_] = data[_]
              }
            })
            console.log(`* DTI ADJUSTER: Restored cache`)
          }
        }
        this.cacheProcessed = true
      },
      clear() {
        Object.keys(this.data || {}).forEach(_ => {
          if (typeof this.data[_] === 'object') {
            this.data[_] = []
          } else {
            this.data[_] = null
          }
        })
        localStorage.removeItem(CACHE_KEY)
      },
      copy() {
        if (!this.copied) {
          const doc = document;
          const text = doc.getElementById('copy-area');
          let range;
          let selection;

          if(doc.body.createTextRange) {

              range = doc.body.createTextRange();
              range.moveToElement(text);
              range.select();

          } else if (window.getSelection) {

              selection = window.getSelection();

              range = doc.createRange();
              range.selectNodeContents(text);

              selection.removeAllRanges();
              selection.addRange(range);

          }

          document.execCommand('copy');
          window.getSelection().removeAllRanges();
          this.copied = true
          setTimeout(() => {
            this.copied = false
          }, 700);
        }
      }
    }
  }
</script>

<style scoped lang="scss">
</style>