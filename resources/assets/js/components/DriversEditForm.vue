<template>
  <validator name="validation">
    <div class="form-horizontal">

      <!-- Driver Name -->
      <div class="form-group"
        :class="{ 'has-error': $validation.name.invalid && ($validation.name.touched || showErrors) }"
      >
        <label for="name" class="col-sm-2">Name</label>
        <div class="col-sm-10">
          <input type="text" class="form-control" placeholder="Driver name"
            v-model="driver.name"
            v-validate:name="{
              required: { rule: true },
              pattern: { rule: '/\\w{2,}/' },
              maxlength: { rule: 20 },
              unique: { rule: true, initial: 'off' }
            }"
          >
          <span v-if="checking" class="help-block">checking...</span>
          <div v-if="showErrors || $validation.name.touched">
            <span v-if="$validation.name.required" class="help-block">This field is required</span>
            <span v-if="$validation.name.pattern" class="help-block">Please enter at least 2 characters</span>
            <span v-if="$validation.name.maxlength" class="help-block">Please enter at most 20 characters</span>
            <span v-if="$validation.name.unique" class="help-block">This name is not available</span>
          </div>
        </div>
      </div>

      <!-- Driver Country -->
      <div class="form-group"
        :class="{ 'has-error': $validation.country.invalid && ($validation.country.touched || showErrors) }"
      >
        <label for="date" class="col-sm-2">Country</label>
        <div class="col-sm-6">
          <select class="form-control"
            v-model="driver.country"
            v-validate:country="{ minlength: 1 }"
          >
            <option v-if="$route.name === 'drivers.create'" disabled selected value="">Select Country</option>
            <option v-for="country in countries" value="{{ country.ccode }}">{{ country.cname }}</option>
          </select>
          <div v-if="showErrors || $validation.country.touched">
            <span v-if="$validation.country.minlength" class="help-block">This field is required</span>
          </div>
        </div>
      </div>

      <!-- Driver Month -->
      <div class="form-group"
        :class="{ 'has-error': $validation.name.invalid && ($validation.name.touched || showErrors) }"
      >
        <label for="name" class="col-sm-2">Birthday Month</label>
        <div class="col-sm-10">
          <input type="number" min="0" max="12" class="form-control" placeholder="month"
            v-model="driver.month"
            v-validate:month="{ required: true, min: 0, max: 12 }"
          >
          <div v-if="showErrors || $validation.name.touched">
            <span v-if="$validation.name.min" class="help-block">Please enter a number between 0 and 12</span>
            <span v-if="$validation.name.max" class="help-block">Please enter a number between 0 and 12</span>
          </div>
        </div>
      </div>

      <!-- Driver Photo -->
      <div class="form-group" :class="{ 'has-error': photoError }">
        <label for="photo" class="col-sm-2">Photo</label>
        <div class="col-sm-10">
          <button class="btn btn-default btn-sm"
            @click="driver.photo = ''"
            v-show="driver.photo"
          >
            Remove photo
          </button>
          <input type="file" class="form-control" id="photo-upload"
            v-else
            @change="validatePhoto"
          >
          <span class="help-block">{{ photoError }}</span>
        </div>
      </div>

      <!-- Buttons -->
      <div class="col-sm-10 col-sm-offset-2">
        <button class="btn btn-primary"
          :class="{ 'disabled': $validation.invalid }"
          @click="save"
        >
          Save
        </button>
        <button @click="cancel" class="btn btn-default">Cancel</button>
      </div>

    </div>
  </validator>
</template>

<script>
  import R from 'ramda'
  import driver from '../api/driver'
  import { getCurrentDriver } from '../vuex/drivers/getters'
  import { createDriver, updateDriver } from '../vuex/drivers/actions'
  import countries from '../utilities/countries'

  export default {
    vuex: {
      actions: {
        createDriver,
        updateDriver
      },
      getters: {
        currentDriver: getCurrentDriver
      }
    },
    validators: {
      unique (val) {
        this.vm.checking = true
        return driver.checkName(val.trim()).then(this.vm.checking = false)
          .catch(json => Promise.reject(new ValidationError(json.message)))
      }
    },
    props: ['show'],
    data () {
      return {
        driver: { name: '', country: '', month: '', photo: '' },
        countries: countries,
        checking: false,
        formAttempted: false,
        photoError: ''
      }
    },
    computed: {
      showErrors () {
        return (this.formAttempted && this.$validation.invalid) ? true : false
      },
      formInvalid () {
        return this.$validation.invalid
      }
    },
    methods: {
      validatePhoto () {
        let photo = document.getElementById('photo-upload').files[0]

        this.photoError = ''

        if (!photo) return false
        if (photo && !photo.type.match('image.*')) {
          this.photoError = 'Uploaded file must be an image'
          return false
        }
        if (photo && photo.size > 2048000) {
          this.photoError = 'Maximum file size is 2MB'
          console.log(this.photoError)
          return false
        }

        return true
      },
      save () {
        if (this.formInvalid) {
          this.formAttempted = true
          return false
        }

        let photo = document.getElementById('photo-upload').files[0]
        let formData = new FormData()

        formData.append('name', this.driver.name.trim())
        formData.append('country', this.driver.country)
        formData.append('month', this.driver.month)
        if (this.validatePhoto()) formData.append('photo', photo, photo.name)
        else formData.append('photo', this.driver.photo)

        if (this.$route.name === 'drivers.create') {
          this.createDriver(formData).then(() => this.$router.go({ name: 'drivers.index' }))
        }
        if (this.$route.name === 'drivers.show') {
          // Laravel cannot handle patch request with files
          formData.append('_method', 'patch')

          this.updateDriver(this.driver.id, formData).then(() => this.show = false)
        }
      },
      cancel () {
        if (this.$route.name === 'drivers.create') return this.$router.go({ name: 'drivers.index' })
        if (this.$route.name === 'drivers.show') return this.show = false
      }
    },
    created () {
      this.driver = R.clone(this.currentDriver)
    }
  }
</script>
