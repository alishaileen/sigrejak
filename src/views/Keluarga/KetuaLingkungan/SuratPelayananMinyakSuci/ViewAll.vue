<template>
  <div>
    <btn-kembali path="/keluarga/ketua/surat" />
    
    <h1>Surat Pelayanan Minyak Suci</h1>

    <div class="data-table mt-5">
      <v-card flat>
        <v-data-table
          :headers="headers"
          :items="suratNotDeleted"
          :search="search"
          :page.sync="page"
          :items-per-page="selectedJumlahData"
          :loading="tableLoading"
          loading-text="Memuat data ..."
          hide-default-footer
          @page-count="pageCount = $event"
        >
          <!-- TABLE TOP -->
          <template v-slot:top>
            <v-card-title>
              <v-text-field
                v-model="search"
                prepend-inner-icon="mdi-magnify"
                label="Cari"
                single-line
                hide-details
                outlined
                dense
                background-color="#FAFAFA"
              ></v-text-field>
              <v-tooltip top>
                <template v-slot:activator="{ on, attrs }">
                <v-btn
                  v-bind="attrs"
                  v-on="on"
                  class="btn text-none ml-4 mt-2"
                  color="blue accent-4"
                  tag="router-link"
                  to="surat-minyak-suci/tambah"
                  dark
                  depressed
                >
                  Buat surat
                </v-btn>
                </template>
                <span class="text-none color-white">Khusus untuk umat non-Kumetiran</span>
              </v-tooltip>
            </v-card-title>
          </template>

          <!-- TABLE CONTENT -->
          <template v-slot:[`item.status_ketua_lingkungan`]="{ item }">
            <approval-table-icon :approval="item.ketua_lingkungan_approval" />
          </template>
          <template v-slot:[`item.action`]="{ item }">
            <div>
              <v-menu bottom offset-y>
                <template v-slot:activator="{ on, attrs }">
                  <v-btn text fab small v-bind="attrs" v-on="on">
                    <v-icon color="#131313">mdi-dots-vertical</v-icon>
                  </v-btn>
                </template>
                <v-list>
                  <v-list-item @click="goToDetail(item.id)">
                    <v-list-item-title>Detail</v-list-item-title>
                  </v-list-item>
                  <v-list-item :disabled="item.ketua_lingkungan_approval === 1" @click="openConfirmDelete(item.id)">
                    <v-list-item-title>Hapus</v-list-item-title>
                  </v-list-item>
                </v-list>
              </v-menu>
            </div>
          </template>

          <template v-slot:footer>
            <v-divider></v-divider>
            <div class="d-flex justify-start text-center py-2">
              <v-pagination
                class="table-pagination ml-2 mb-3"
                v-model="page"
                :length="pageCount"
                :total-visible="6"
                color="blue accent-4"
              ></v-pagination>
            </div>
          </template>
        </v-data-table>
      </v-card>
    </div>

    <snackbar></snackbar>

    <confirm-delete-modal
      @confirmDelete="confirmDeleteData"
    ></confirm-delete-modal>
  </div>
</template>

<script>
import { getData, editData } from '@/utils'

import ApprovalTableIcon from '@/components/ApprovalTableIcon'

export default {
  components: {
    ApprovalTableIcon,
  },
  data: () => ({
    url: '/surat-pelayanan-minyak-suci',
    tableLoading: true,
    search: '',
    headers: [
      {
        text: 'Tgl surat', value: 'created_at',
      },
      {
        text: 'Nama', value: 'nama',
      },
      {
        text: 'Keluarga', value: 'nama_keluarga_penanggung_jawab',
      },
      {
        text: 'No. telp keluarga', value: 'no_telp_keluarga_penanggung_jawab',
      },
      {
        text: 'K. Lingkungan', value: 'status_ketua_lingkungan', align: 'center', sortable: false
      },
      {
        text: 'Aksi', value: 'action', sortable: false
      },
    ],
    surat: [],
    page: 1,
    pageCount: 0,
    selectedJumlahData: 10,
    jumlahData: [10, 30, 50],
    deleteId: null,
    isModalDetailActive: false,
    selectedDetail: null,
  }),
  computed: {
    suratNotDeleted() {
      return this.surat.filter(e => e.deleted_at === null)
    }
  },
  async mounted() {
    this.tableLoading = true
    this.surat = await getData(`${this.url}/lingkungan/${this.$store.state.keluarga.lingkunganId}`)
    this.tableLoading = false
  },
  methods: {
    goToDetail(id) {
      this.$router.push(`surat-minyak-suci/detail/${id}`)
    },
    openConfirmDelete(id) {
      this.deleteId = id
      this.$store.dispatch('deleteData/openModal')
    },
    async confirmDeleteData(decision) {
      // Close confirmation modal
      this.$store.commit('deleteData/resetModal')
      
      if (decision) {
        let snackbar = {}

        // Activate loading overlay
        this.$store.dispatch('loading/openLoading')

        try {
          let response = await editData(this.url, this.deleteId)
          
          if (response.status === 200) {
            snackbar.color = 'success'
            snackbar.text = 'Data berhasil dihapus'
            this.surat = await getData(`/surat-keterangan/keluarga/${this.$store.state.keluarga.id}`)
          } else {
            snackbar.color = 'error'
            snackbar.text = 'Terjadi kesalahan. Silahkan refresh dan coba lagi'
          }
        } catch (error) {
          snackbar.color = 'error'
          snackbar.text = error
        }
        this.$store.dispatch('snackbar/openSnackbar', snackbar)
        this.$store.dispatch('loading/closeLoading')
      }
    }
  }
}
</script>