<template>
  <div>
    <btn-kembali path="/keluarga/surat/surat-komuni-penguatan" />

    <h1>Edit Surat {{ formData.jenis_surat === 1 ? 'Komuni I' : 'Penguatan' }}</h1>

    <div class="form mt-5">
      <v-card class="mx-auto" flat>
        <v-card-title>
          <h3>Detail Informasi</h3>
          
          <v-spacer></v-spacer>

          <approval-chip
            :approval="formData.ketua_lingkungan_approval"
            role="Ketua Lingkungan"
            :nama="formData.ketua_lingkungan"
          ></approval-chip>

          <approval-chip
            :approval="formData.sekretariat_approval"
            role="Sekretariat"
            :nama="sekretariat.nama"
          ></approval-chip>

          <button-chat
            :countChatUnread="countChatUnread"
            :chatPageUrl="`/keluarga/surat/surat-komuni-penguatan/chat/${formData.id}`"
            :detailPageUrl="`/keluarga/surat/surat-komuni-penguatan/detail/${formData.id}`"
            :endpointUrl="url"
          ></button-chat>
        </v-card-title>

        <v-divider></v-divider>

        <v-form class="pa-6" ref="form" @submit.prevent="submit">
          <div class="mb-15">
            <label>No. surat</label>
            <p>
              {{ formData.no_surat }}
            </p>

            <label>Jenis surat</label>
            <p>
              {{ formData.jenis_surat === 1 ? 'Komuni I' : 'Penguatan' }}
            </p>

            <v-btn
              class="text-none"
              depressed
              color="blue"
              text
              outlined
              @click="isSidebarLogActive = true"
            >
              Log surat
            </v-btn>
          </div>
          
          <h3 class="mb-5">Informasi Umat</h3>

          <autocomplete
            label="Nama*"
            :value="formData.nama"
            :suggestionList="anggotaKeluarga"
            itemText="nama"
            @changeData="changeIdUmat"
            :disable="(!isEditable)"
            :rules="[required]"
          ></autocomplete>

          <v-alert
            v-show="isAlertNotBaptized"
            border="left"
            colored-border
            type="error"
            elevation="2"
          >
            <span>
              Umat yang dipilih belum memiliki catatan nama baptis di sistem.
              Pastikan bahwa umat sudah dibaptis.
            </span>
          </v-alert>

          <label>Nama baptis</label>
          <p>{{ formData.nama_baptis }}</p>

          <label>Tempat lahir</label>
          <p>{{ formData.tempat_lahir }}</p>

          <label>Tanggal lahir</label>
          <p>{{ formData.tgl_lahir }}</p>

          <label>Nomor telp/HP</label>
          <p>{{ formData.no_telp }}</p>

          <label>Paroki tempat baptis*</label>
          <v-text-field
            v-model="formData.paroki_baptis"
            outlined
            dense
            :disabled="(!isEditable)"
            :readonly="(!isEditable)"
            :rules="[required]"
          ></v-text-field>

          <label>Nomor surat baptis*</label>
          <v-text-field
            v-model="formData.no_surat_baptis"
            required
            outlined
            dense
            :disabled="(!isEditable)"
            :readonly="(!isEditable)"
            :rules="[required]"
          ></v-text-field>

          <label>Tanggal baptis*</label>
          <v-menu
            ref="menuTglBaptis"
            v-model="isDatePickerTglBaptisActive"
            :close-on-content-click="false"
            transition="scale-transition"
            offset-y
            min-width="auto"
            :disabled="(!isEditable)"
          >
            <template v-slot:activator="{ on, attrs }">
              <v-text-field
                v-model="formData.tgl_baptis"
                prepend-inner-icon="mdi-calendar"
                readonly
                outlined
                dense
                v-bind="attrs"
                v-on="on"
                :disabled="(!isEditable)"
                :rules="[required]"
              ></v-text-field>
            </template>
            <v-date-picker
              v-model="formData.tgl_baptis"
              :max="new Date().toISOString().substr(0, 10)"
              @change="(date) => { this.$refs.menuTglBaptis.save(date) }"
              :disabled="(!isEditable)"
              :readonly="(!isEditable)"
            ></v-date-picker>
          </v-menu>

          <label>Nama sekolah*</label>
          <v-text-field
            v-model="formData.sekolah"
            required
            outlined
            dense
            :disabled="(!isEditable)"
            :readonly="(!isEditable)"
            :rules="[required]"
          ></v-text-field>

          <label>Kelas*</label>
          <v-text-field
            v-model="formData.kelas"
            required
            outlined
            dense
            :disabled="(!isEditable)"
            :readonly="(!isEditable)"
            :rules="[required]"
          ></v-text-field>

          <v-divider class="mb-5"></v-divider>

          <h3 class="mb-5">Informasi Orang Tua</h3>

          <v-alert
            v-show="isAlertOrtuActive"
            border="left"
            colored-border
            type="error"
            elevation="2"
          >
            <span>
              Anak yang dipilih memiliki catatan orang tua yang belum lengkap.
              Harap lengkapi informasi di menu Anggota Keluarga.
            </span>
          </v-alert>

          <div v-show="!isAlertOrtuActive">
            <label>Nama ayah*</label>
            <p>{{ formData.nama_ayah }}</p>
            
            <label>Nama ibu*</label>
            <p>{{ formData.nama_ibu }}</p>
          </div>
          
          <v-divider class="mb-5"></v-divider>

          <h3 class="mb-5">Informasi Wali Penguatan</h3>

          <label>Nama santo/a pelindung*</label>
          <v-text-field
            v-model="formData.nama_pelindung"
            required
            outlined
            dense
            :disabled="(!isEditable)"
            :readonly="(!isEditable)"
            :rules="[required]"
          ></v-text-field>

          <label>Nama Wali Penguatan*</label>
          <v-text-field
            v-model="formData.nama_wali_penguatan"
            required
            outlined
            dense
            :disabled="(!isEditable)"
            :readonly="(!isEditable)"
            :rules="[required]"
          ></v-text-field>

          <label>Tanggal krisma wali*</label>
          <v-menu
            ref="menuTglKrisma"
            v-model="isDatePickerTglKrismaWaliActive"
            :close-on-content-click="false"
            transition="scale-transition"
            offset-y
            min-width="auto"
            :disabled="(!isEditable)"
          >
            <template v-slot:activator="{ on, attrs }">
              <v-text-field
                v-model="formData.tgl_krisma_wali"
                prepend-inner-icon="mdi-calendar"
                readonly
                outlined
                dense
                v-bind="attrs"
                v-on="on"
                :disabled="(!isEditable)"
                :rules="[required]"
              ></v-text-field>
            </template>
            <v-date-picker
              v-model="formData.tgl_krisma_wali"
              :max="new Date().toISOString().substr(0, 10)"
              @change="(date) => { this.$refs.menuTglKrisma.save(date) }"
              :disabled="(!isEditable)"
              :readonly="(!isEditable)"
            ></v-date-picker>
          </v-menu>

          <label>File syarat*</label>
          <div class="my-5" v-if="(!editFileSyarat)">
            <v-btn v-if="formData.file_syarat && (typeof formData.file_syarat) == 'string'"
              text
              small
              color="blue"
              @click="downloadFile(formData.file_syarat)"
            >
              Klik untuk melihat file
            </v-btn>
            <v-btn
              v-if="isEditable"
              class="text-none"
              color="orange"
              dark
              depressed
              @click="editFileSyarat = true"
            >
              Ubah file
            </v-btn>
          </div>
          <div class="d-flex mb-5" v-if="editFileSyarat">
            <v-file-input
              accept="application/zip"
              style="display: none;"
              ref="inputSyarat"
              v-model="formData.file_syarat"
              :rules="[required,acceptZipOnly]"
            ></v-file-input>
            <div>
              <v-btn
                v-if="isEditable"
                class="text-none"
                color="blue darken-3"
                dark
                depressed
                @click="$refs.inputSyarat.$refs.input.click()"
              >
                <v-icon>mdi-upload</v-icon>
                Upload file
              </v-btn>
            </div>
            <div v-if="formData.file_syarat && (typeof formData.file_syarat) != 'string'" class="ml-5">
              <p class="ma-0">{{formData.file_syarat.name}}</p>
              <small>{{ fileSize }} Mb</small>
            </div>
          </div>

          <div class="d-flex justify-end">
            <v-btn
              class="btn text-none mt-2"
              type="submit"
              color="blue accent-4"
              dark
              depressed
              :disabled="isSubmitDisabled"
            >
              Simpan
            </v-btn>
          </div>
        </v-form>
      </v-card>     
    </div>
    <snackbar />

    <sidebar-log-surat
      :logList="logList"
      :isSidebarActive="isSidebarLogActive"
      @closeSidebar="isSidebarLogActive = false"
    ></sidebar-log-surat>
  </div>
</template>

<script>
import { API_URL } from '@/constants'
import { getAnggotaKeluargaNotDeleted, getOneData, getLogSuratByNoSurat, editData, changeDateFormat } from '@/utils'
import Autocomplete from '@/components/Autocomplete'
import ApprovalChip from '@/components/ApprovalChip.vue'
import SidebarLogSurat from '@/components/SidebarLogSurat.vue'
import ButtonChat from '@/components/ButtonChat.vue'
import { required, acceptZipOnly } from '@/validations'

export default {
  components: {
    Autocomplete,
    ApprovalChip,
    SidebarLogSurat,
    ButtonChat,
  },
  data: () => ({
    url: '/surat-komuni-penguatan',
    isEditable: false,
    isDatePickerTglBaptisActive: false,
    isDatePickerTglKrismaWaliActive: false,
    editFileSyarat: false,
    formData: { file_syarat: {size: 0}},
    anggotaKeluarga: [],
    isAlertNotBaptized: false,
    isAlertOrtuActive: false,
    sekretariat: { nama: '' },
    logList: [],
    isSidebarLogActive: false,
    countChatUnread: 0,

    // validation rules
    required,
    acceptZipOnly,
  }),
  computed: {
    isSubmitDisabled() {
      return (this.isAlertOrtuActive || this.isAlertNotBaptized || !this.isEditable) ? true : false
    },
    fileSize() {
      let sizeInMb = this.formData.file_syarat.size/1024/1024
      sizeInMb = sizeInMb.toString()
      sizeInMb = sizeInMb.substring(0, 5)

      return sizeInMb
    }
  },
  async mounted() {
    this.anggotaKeluarga = await getAnggotaKeluargaNotDeleted(this.$store.state.keluarga.id)
    
    this.formData = await getOneData(`${this.url}/${this.$route.params.id}`)
    this.formData.tgl_lahir = changeDateFormat(this.formData.tgl_lahir)

    this.tempJenisSurat = this.formData.jenis_surat === 1 ? 'Komuni I' : 'Penguatan'

    // Get Log surat
    this.logList = await getLogSuratByNoSurat(this.formData.id)

    // Get data sekretariat and romo if surat has been approved
    if(this.formData.id_sekretariat != null) {
      this.sekretariat = await getOneData(`/admin/${this.formData.id_sekretariat}`)
    }

    // Set editable boolean to true if ketua lingkungan have not approved
    this.isEditable = this.formData.ketua_lingkungan_approval === 1 ? false : true

    // Get jumlah chat yg belum read
    this.countChatUnread = await getOneData(`/chat/count-unread/${this.$route.params.id}`)
    this.countChatUnread = this.countChatUnread.count_unread
  },
  methods: {
    async downloadFile(fileName) {
      window.open(`${API_URL}/files/${fileName}`, '_blank')
    },

    
    async changeIdUmat(e) {
      let temp = this.anggotaKeluarga.find(_ => {
        return _.nama === e
      })
      this.formData.id_umat = temp.id
      this.formData.id_lingkungan = temp.lingkungan_id
      this.formData.nama_baptis = temp.nama_baptis
      this.formData.no_telp = temp.no_telp
      this.formData.tempat_lahir = temp.tempat_lahir
      this.formData.tgl_lahir = changeDateFormat(temp.tgl_lahir)

      this.isAlertNotBaptized = (temp.nama_baptis === null) ? true : false

      let detailTemp = await getOneData(`/detail-umat/${temp.id}`)

      await this.setOrtu(detailTemp.id_ayah, detailTemp.id_ibu)
    },
    async setOrtu(idAyah, idIbu) {
      let tempOrangTua

      this.isAlertOrtuActive = (!idAyah || !idIbu) ? true : false

      if (idAyah) {
        tempOrangTua = await getOneData(`/umat/${idAyah}`)
        this.formData.nama_ayah = tempOrangTua.nama || '-'
      }
      if (idIbu) {
        tempOrangTua = await getOneData(`/umat/${idIbu}`)
        this.formData.nama_ibu = tempOrangTua.nama || '-'
      }
    },
    async submit() {
      let snackbar = {}
      
      if(!this.$refs.form.validate()) {
        this.$refs.form.validate()
        snackbar.color = 'error',
        snackbar.text = 'Harap periksa inputan anda kembali'
        this.$store.dispatch('snackbar/openSnackbar', snackbar)
        return
      }

      this.$store.dispatch('loading/openLoading')
      this.$store.commit('snackbar/resetSnackbar')

      let formData = new FormData()
      
      if(this.formData.cara_ortu_menikah === 'Cara lain') {
        this.formData.cara_ortu_menikah = this.temp_cara_ortu_menikah
      }
      this.formData.jenis_surat = this.tempJenisSurat === 'Komuni I' ? 1 : 2

      formData.append('id_keluarga', this.formData.id_keluarga)
      formData.append('id_lingkungan', this.formData.id_lingkungan)
      formData.append('jenis_surat', this.formData.jenis_surat)
      formData.append('id_umat', this.formData.id_umat)
      formData.append('paroki_baptis', this.formData.paroki_baptis)
      formData.append('tgl_baptis', this.formData.tgl_baptis)
      formData.append('no_surat_baptis', this.formData.no_surat_baptis)
      formData.append('sekolah', this.formData.sekolah)
      formData.append('kelas', this.formData.kelas)
      formData.append('nama_pelindung', this.formData.nama_pelindung)
      formData.append('nama_wali_penguatan', this.formData.nama_wali_penguatan)
      formData.append('tgl_krisma_wali', this.formData.tgl_krisma_wali)
      if (typeof this.formData.file_syarat != 'string') {
        formData.append('file_syarat', this.formData.file_syarat)
      }

      try {
        let response = await editData(this.url, this.formData.id, formData)

        if (response.status >= 200 && response.status < 300) {
          snackbar.color = 'success',
          snackbar.text = 'Surat berhasil diubah!'
          this.$router.push('/keluarga/surat/surat-komuni-penguatan')
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

<style>

</style>