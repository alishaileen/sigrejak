<template>
  <div>
    <btn-kembali path="/pengurus/keluarga" />
    
    <h1>Tambah Keluarga</h1>

    <div class="form mt-5" @submit.prevent="submit">
      <v-card class="pa-6 mx-auto" flat>
        <v-form>
          <label>Nama keluarga*</label>
          <v-text-field
            v-model="formData.nama_keluarga"
            required
            outlined
            dense
          ></v-text-field>

          <label>Username*</label>
          <v-text-field
            v-model="formData.username"
            required
            outlined
            dense
          ></v-text-field>

          <label>Email*</label>
          <v-text-field
            v-model="formData.email"
            required
            outlined
            dense
          ></v-text-field>

          <label>Nama kepala keluarga*</label>
          <v-text-field
            v-model="formData.nama_kepala_keluarga"
            required
            outlined
            dense
          ></v-text-field>

          <label>No. telepon kepala keluarga*</label>
          <v-text-field
            v-model="formData.no_telp_kepala_keluarga"
            required
            outlined
            dense
          ></v-text-field>

          <small>Password akan di-generate sistem</small>

          <div class="d-flex justify-end">
            <v-btn
              class="btn text-none mt-2"
              type="submit"
              color="blue accent-4"
              dark
              depressed
            >
              Tambah Keluarga
            </v-btn>
          </div>
        </v-form>
      </v-card>
    </div>

    <modal-password
      :isModalActive="isPassModalActive"
      :password="generatedPass"
      @close="closePassModal"
    />

    <snackbar></snackbar>
  </div>
</template>

<script>
import { postData } from '../../../utils'
import ModalPassword from '../../../components/ModalPassword.vue'

export default {
  components: {
    ModalPassword,
  },
  data: () => ({
    formData: {
      nama_keluarga: '',
      username: '',
      email: '',
    },
    isPassModalActive: false,
    generatedPass: '',
  }),
  methods: {
    closePassModal() {
      this.isPassModalActive = false
      this.$router.push('/pengurus/keluarga')
    },
    async submit() {
      this.$store.dispatch('loading/openLoading')
      this.$store.commit('snackbar/resetSnackbar')

      let snackbar = {}

      try {
        let response = await postData('/keluarga/register', this.formData)

        if (response.status >= 200 && response.status < 300) {
          snackbar.color = 'success',
          snackbar.text = 'Data berhasil ditambahkan!'
          this.generatedPass = response.data.result.generatedPassword
          this.isPassModalActive = true
        } else {
          snackbar.color = 'error',
          snackbar.text = 'Harap periksa kembali inputan anda'
        }
      } catch (error) {
        snackbar.color = 'error',
        snackbar.text = error
      }

      this.$store.dispatch('snackbar/openSnackbar', snackbar)
      this.$store.dispatch('loading/closeLoading')
    },
  }
}
</script>