<template>
  <div>
    <h2>{{insuranceDetail.name}}</h2>
    <small class="mute mb">{{insuranceDetail.description}}dd</small>
    <form method="post" @submit.prevent="submit">
      <b-field :label="field.field" v-for="(field, key) in insuranceDetail.field_set"
        :message="field.message" :key="key">
          <b-select :placeholder="field.field" v-if="field.type === 'select'"
            v-model="submission[field.id]" expanded>
              <option v-for="(option, key) in field.option_set" :key="key" :value="option.id">
                {{ option.display }}
              </option>
          </b-select>
          <b-datepicker
              :placeholder="field.field" icon="calendar-today" v-model="submission[field.id]"
              v-else-if="field.type === 'date'">
          </b-datepicker>
          <div class="block" v-else-if="field.type === 'radio'">
            <b-radio  v-for="(option, key) in field.option_set" :key="key"
              v-model="submission[field.id]"  :native-value="option.id">{{option.display}}</b-radio>
          </div>
          <div class="block" v-else-if="field.type === 'checkbox'">
            <b-checkbox  v-for="(option, key) in field.option_set" :key="key"
              v-model="submission[field.id]"  :native-value="option.id">{{option.display}}</b-checkbox>
          </div>

          <b-input v-model="submission[field.id]" :pattern="field.validation" :title="field.message"
            :type="field.type" :placeholder="field.field" :required="field.is_required" v-else></b-input>
      </b-field>
      <button class="button is-primary">Submit</button>
    </form>
  </div>
</template>

<script>
export default {
  name: 'SubmitRiskForm',
  data () {
    return {
      isRequired: true,
      insuranceDetail: {},
      submission: {}
    }
  },
  created () {
    const loader = this.$loading.open()
    this.$axios.get(`${this.$api}insurer/risks/${this.$route.params.id}/`).then(data => {
      loader.close()
      for (const field of data.data['field_set']) {
        if (field.type === 'checkbox') {
          this.submission[field.id] = []
        }
      }
      this.insuranceDetail = data.data
    }).catch(err => {
      loader.close()
      this.$toast.open({
        duration: 5000,
        message: `Unable to fetch risk informaiton<br/>${err.message}`,
        position: 'is-bottom',
        type: 'is-danger'
      })
      this.$router.push('/')
    })
  },
  methods: {
    submit () {
      const loader = this.$loading.open()
      const params = new URLSearchParams()
      params.append('risk', this.$route.params.id)
      const fields = []
      for (const key in this.submission) {
        fields.push({
          'field': key,
          'value': this.submission[key]
        })
      }
      params.append('submissionvalue_set', fields)
      this.$axios.post(`${this.$api}insurer/submit/`, {
        'risk': this.$route.params.id,
        'submissionvalue_set': fields
      }).then(response => {
        this.$toast.open({
          message: 'Form has been submitted!',
          type: 'is-success',
          position: 'is-bottom'
        })
        this.submission = {}
        loader.close()
      }).catch(err => {
        loader.close()
        this.$toast.open({
          duration: 5000,
          message: `Error occured while creating risk<br/>${err.message}`,
          position: 'is-bottom',
          type: 'is-danger'
        })
      })
    }
  }
}
</script>
