<template>
  <div>
    <btn-kembali path="/keluarga/anggota" />
    
    <h1>Detail Anggota Keluarga</h1>
    
    <v-card class="form mt-5 pa-6" light flat>
      <h2>Informasi Umum</h2>
      
      <v-form class="mt-4" ref="formUmat" @submit.prevent="saveUmat">
        <v-row>
          <v-col>
            <label>Nama lengkap*</label>
            <v-text-field
              v-model="umat.nama"
              required
              outlined
              dense
              :rules="[required]"
            ></v-text-field>

            <label>Nama baptis</label>
            <p>{{ umat.nama_baptis || '-' }}</p>

            <label>Jenis kelamin*</label>
            <v-select
              :items="[ 'Pria', 'Wanita' ]"
              v-model="umat.jenis_kelamin"
              outlined
              dense
              :rules="[required]"
            ></v-select>

            <label>Tempat lahir*</label>
            <v-text-field
              v-model="umat.tempat_lahir"
              required
              outlined
              dense
              :rules="[required]"
            ></v-text-field>

            <label>Tanggal lahir*</label>
            <birth-date-picker
              :tgl="umat.tgl_lahir"
              @saveDate="saveDate"
              :rules="[required]"
            ></birth-date-picker>
          </v-col>

          <v-col>
            <label>Nomor telepon</label>
            <v-text-field
              v-model="umat.no_telp"
              required
              outlined
              dense
            ></v-text-field>

            <label>Pekerjaan*</label>
            <v-text-field
              v-model="umat.pekerjaan"
              required
              outlined
              dense
              :rules="[required]"
            ></v-text-field>

            <label>Alamat*</label>
            <v-text-field
              v-model="umat.alamat"
              required
              outlined
              dense
              :rules="[required]"
            ></v-text-field>

            <autocomplete
              :value="umat.nama_lingkungan"
              label="Lingkungan tempat tinggal*"
              :suggestionList="lingkunganList"
              itemText="nama_lingkungan"
              @changeData="changeIdLingkungan"
              :rules="[required]"
            ></autocomplete>
          </v-col>
        </v-row>

        <div class="d-flex justify-end">
          <v-btn
            class="btn text-none"
            color="blue accent-4"
            dark
            depressed
            type="submit"
          >
            Simpan
          </v-btn>
        </div>
      </v-form>

      <h2>Informasi Lain</h2>

      <v-divider class="mt-4"></v-divider>

      <v-banner>
        <v-avatar
          slot="icon"
          color="yellow darken-3"
          size="35"
        >
          <v-icon
            icon="mdi-exclamation-thick"
            color="white"
            small
          >
            fas fa-exclamation
          </v-icon>
        </v-avatar>
          <span>
            Hanya data ayah, ibu, file akta lahir, dan file ktp yang dapat diubah.
            <br>
            Jika data salah, harap hubungi sekretariat
          </span>
      </v-banner>
      
      <v-form class="mt-4" enctype="multipart/form-data" ref="formDetail" @submit.prevent="saveDetailUmat">
        <v-row>
          <v-col>
            <autocomplete
              :value.sync="namaAyah"
              :disable="false"
              label="Ayah*"
              :suggestionList="anggotaKeluargaNotDeleted"
              itemText="nama"
              @changeData="changeIdAyah"
              :rules="[required]"
            ></autocomplete>
          </v-col>
          <v-col>
            <autocomplete
              :value="namaIbu"
              :disable="false"
              label="Ibu*"
              :suggestionList="anggotaKeluargaNotDeleted"
              itemText="nama"
              @changeData="changeIdIbu"
              :rules="[required]"
            ></autocomplete>
          </v-col>
        </v-row>

        <div class="mb-5">
          <label>File akta lahir*</label>
          <v-file-input
            style="display: none"
            show-size
            ref="inputAktaLahir"
            accept="image/*"
            v-model="detailUmat.file_akta_lahir"
            @change="previewImage('akta')"
            :rules="[required]"
          ></v-file-input>
          <div>
            <v-btn
              class="text-none"
              color="blue darken-3"
              dark
              depressed
              @click="selectFile('akta')"
            >
              <v-icon>mdi-upload</v-icon>
                Upload file
            </v-btn>
          </div>
          <div class="image-preview" v-if="detailUmat.file_akta_lahir != null">
            <v-img contain :src="displayGambarAkta"></v-img>
          </div>
        </div>

        <div class="mb-5">
          <label>File KTP</label>
          <v-file-input
            style="display: none"
            show-size
            ref="inputKtp"
            accept="image/*"
            v-model="detailUmat.file_ktp"
            @change="previewImage($event, 'ktp')"
          ></v-file-input>
          <div>
            <v-btn
              class="text-none"
              color="blue darken-3"
              dark
              depressed
              @click="selectFile('ktp')"
            >
              <v-icon>mdi-upload</v-icon>
                Upload file
            </v-btn>
          </div>
          <div class="image-preview" v-if="detailUmat.file_ktp != null">
            <v-img contain :src="displayGambarKtp"></v-img>
          </div>
        </div>

        <label>Tanggal baptis</label>
        <p>{{ detailUmat.tgl_baptis ? detailUmat.tgl_baptis : '-' }}</p>

        <label>Tanggal komuni</label>
        <p>{{ detailUmat.tgl_komuni ? detailUmat.tgl_komuni : '-' }}</p>

        <label>Tanggal penguatan</label>
        <p>{{ detailUmat.tgl_penguatan ? detailUmat.tgl_penguatan : '-' }}</p>
        
        <div class="d-flex justify-end">
          <v-btn
            class="btn text-none"
            color="blue accent-4"
            dark
            depressed
            type="submit"
          >
            Simpan
          </v-btn>
        </div>
      </v-form>
    </v-card>
    <snackbar></snackbar>
  </div>
</template>

<script>
// import axios from 'axios'
import { getData, getAnggotaKeluargaNotDeleted, getOneData, editData } from '../../../utils'
import { API_URL } from '../../../constants'

import { required } from '@/validations'

import BirthDatePicker from '../../../components/BirthDatePicker'
import Autocomplete from '../../../components/Autocomplete'

export default {
  components: {
    BirthDatePicker,
    Autocomplete
  },
  data: () => ({
    lingkunganList: [],
    umat: {},
    detailUmat: {},
    anggotaKeluarga: [],
    namaAyah: '',
    namaIbu: '',

    displayGambarAkta: null,
    displayGambarKtp: null,

    // validation rules
    required,
  }),
  computed: {
    anggotaKeluargaNotDeleted() {
      return this.anggotaKeluarga.filter(e => e.id != this.umat.id && e.deleted_at === null)
    },
  },
  async mounted() {
    this.lingkunganList = await getData(`/lingkungan`)
    this.anggotaKeluarga = await getAnggotaKeluargaNotDeleted(this.$store.state.keluarga.id)

    this.umat = await getOneData(`/umat/${this.$route.params.id}`)
    this.detailUmat = await getOneData(`/detail-umat/${this.$route.params.id}`)
    
    if(this.detailUmat.id_ayah) this.setNamaAyah()
    if(this.detailUmat.id_ibu) this.setNamaIbu()

    if(this.detailUmat.file_akta_lahir) {
      this.getImage(this.detailUmat.file_akta_lahir, 'akta')
    }
    // else {
    //   this.detailUmat.file_akta_lahir = {}
    // }

    if(this.detailUmat.file_ktp) {
      this.getImage(this.detailUmat.file_ktp, 'ktp')
    }
    // else {
    //   this.detailUmat.file_ktp = {}
    // }
  },
  methods: {
    changeIdLingkungan(e) {
      let temp = this.lingkunganList.find(_ => _.nama_lingkungan == e)
      this.umat.lingkungan_id = temp.id
    },
    changeIdAyah(e) {
      let temp = this.anggotaKeluarga.find(_ => _.nama === e)
      this.detailUmat.id_ayah = temp.id
    },
    changeIdIbu(e) {
      let temp = this.anggotaKeluarga.find(_ => _.nama === e)
      this.detailUmat.id_ibu = temp.id
    },
    setNamaAyah() {
      let temp = this.anggotaKeluarga.find(_ => _.id === this.detailUmat.id_ayah)
      
      this.namaAyah = temp ? temp.nama : ''
    },
    setNamaIbu() {
      let temp = this.anggotaKeluarga.find(_ => _.id === this.detailUmat.id_ibu)
      
      this.namaIbu = temp ? temp.nama : ''
    },
    saveDate(newDate) {
      this.umat.tgl_lahir = newDate
    },
    async saveUmat() {
      let snackbar = {}

      if(!this.$refs.formUmat.validate()) {
        this.$refs.formUmat.validate()
        snackbar.color = 'error',
        snackbar.text = 'Harap periksa inputan anda kembali'
        this.$store.dispatch('snackbar/openSnackbar', snackbar)
        return
      }

      this.$store.dispatch('loading/openLoading')
      this.$store.commit('snackbar/resetSnackbar')

      try {
        let response = await editData('/umat', this.$route.params.id, this.umat)

        if (response.status >= 200 && response.status < 300) {          
          snackbar.color = 'success',
          snackbar.text = 'Data berhasil tersimpan!'
          this.umat = await getData(`/umat/${this.$route.params.id}`)
          this.umat = this.umat[0]
        } else {
          snackbar.color = 'error'
          snackbar.text = 'Harap periksa kembali inputan anda'
        }
      } catch (error) {
        snackbar.color = 'error'
        snackbar.text = error
      }

      this.$store.dispatch('snackbar/openSnackbar', snackbar)
      this.$store.dispatch('loading/closeLoading')
    },


    async saveDetailUmat() {
      let snackbar = {}

      if(!this.$refs.formDetail.validate()) {
        this.$refs.formDetail.validate()
        snackbar.color = 'error',
        snackbar.text = 'Harap periksa inputan anda kembali'
        this.$store.dispatch('snackbar/openSnackbar', snackbar)
        return
      }

      this.$store.dispatch('loading/openLoading')
      this.$store.commit('snackbar/resetSnackbar')

      let formData = new FormData()

      if(this.detailUmat.id_ayah)
        formData.append('id_ayah', this.detailUmat.id_ayah)
      if(this.detailUmat.id_ibu)
        formData.append('id_ibu', this.detailUmat.id_ibu)
      
      // if file_akta_lahir bukan string maka append data
      if(this.detailUmat.file_akta_lahir && typeof this.detailUmat.file_akta_lahir != 'string') {
        formData.append('file_akta_lahir', this.detailUmat.file_akta_lahir)
      }
      if(this.detailUmat.file_ktp && typeof this.detailUmat.file_ktp != 'string') {
        formData.append('file_ktp', this.detailUmat.file_ktp)
      }

      try {
        let response = await editData('/detail-umat', this.$route.params.id, formData)

        if (response.status >= 200 && response.status < 300) {          
          snackbar.color = 'success',
          snackbar.text = 'Data berhasil tersimpan!'
          this.detailUmat = await getData(`/detail-umat/${this.$route.params.id}`)
          this.detailUmat = this.detailUmat[0]
        } else {
          snackbar.color = 'error'
          snackbar.text = 'Harap periksa kembali inputan anda'
        }
      } catch (error) {
        snackbar.color = 'error'
        snackbar.text = error
      }

      this.$store.dispatch('snackbar/openSnackbar', snackbar)
      this.$store.dispatch('loading/closeLoading')
    },


    async getImage(endpoint, expectedImage) {
      const url = `${API_URL}/files/${endpoint}`
      // const res = await axios.get(url, { responseType: "blob", });

      // const displayImage = URL.createObjectURL(res.data);
      // const fileName =
      //   expectedImage == "akta"
      //     ? `${this.detailUmat.file_akta_lahir}`
      //     : `${this.detailUmat.file_ktp}`;
      // const imgFile = new File([res.data], fileName, {
      //   type: "image/*",
      // });
      if (expectedImage == "akta") {
        this.displayGambarAkta = url;
        // this.displayGambarAkta = displayImage;
        // this.detailUmat.file_akta_lahir = imgFile;
      } else {
        this.displayGambarKtp = url;
        // this.displayGambarKtp = displayImage;
        // this.detailUmat.file_ktp = imgFile;
      }
    },
    selectFile(file) {
      if (file === 'akta')
        this.$refs.inputAktaLahir.$refs.input.click()
      else if(file === 'ktp')
        this.$refs.inputKtp.$refs.input.click()
    },
    previewImage(img) {
      // Reference to the DOM input element
      let input
      if (img == "akta") {
        input = this.detailUmat.file_akta_lahir;
      } else {
        input = this.detailUmat.file_ktp;
      }
      // Ensure that you have a file before attempting to read it
      if (input) {
        // create a new FileReader to read this image and convert to base64 format
        var reader = new FileReader();
        reader.readAsDataURL(input);
        // Define a callback function to run, when FileReader finishes its job
        reader.onload = (e) => {
          // Note: arrow function used here, so that "this.imageData" refers to the imageData of Vue component
          // Read image as base64 and set to imageData
          if (img == "akta") {
            this.displayGambarAkta = e.target.result;
          } else {
            this.displayGambarKtp = e.target.result;
          }
        };
        // Start the reader job - read file as a data url (base64 format)
        reader.readAsDataURL(input);
      }
    },
  }
}
</script>

<style>

</style>