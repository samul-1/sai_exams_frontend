<template>
  <div
    :class="{ 'bg-gray-100': exam.draft || old }"
    class="flex flex-col w-full px-4 py-4 my-3 mt-auto space-y-2 transition-shadow duration-75 border rounded-lg md:px-2 md:space-y-0 md:flex-row hover:shadow-md"
  >
    <Spinner v-if="loading" :loadingMessage="loadingMessage"></Spinner>
    <h1
      class="max-w-xs mb-4 mr-2 md:my-auto"
      :title="exam.name"
      v-html="truncateString(exam.name, 60)"
    ></h1>
    <!-- left buttons -->
    <div class="grid grid-cols-2 gap-2 md:gap-1 md:flex md:items-center">
      <router-link v-if="notStartedNotClosed" :to="`/editor/${exam.id}`"
        ><button
          :disabled="exam.locked_by"
          class="w-full px-2 py-1 text-sm font-light text-white bg-indigo-700 rounded-lg disabled:opacity-40 hover:bg-indigo-800"
        >
          <i class="mr-1 fas fa-edit "></i>

          Modifica
        </button></router-link
      >
      <router-link v-if="startedNotClosed" :to="`/exams/${exam.id}/progress`"
        ><button
          class="w-full px-2 py-1 text-sm font-light text-white bg-indigo-700 rounded-lg disabled:opacity-40 hover:bg-indigo-800"
        >
          <i class="mr-1 text-xs fas fa-eye"></i>
          Monitora
        </button></router-link
      >
      <button
        @click="confirmClosure(exam)"
        v-if="startedNotClosed"
        class="px-2 py-1 text-sm text-white bg-red-800 rounded-lg disabled:opacity-40 hover:bg-red-900"
      >
        <i class="mr-1 text-xs fas fa-exclamation-triangle"></i> Chiudi
      </button>

      <div v-if="notStartedOrClosed" class="inline-block dropdown">
        <div class="relative">
          <div class="absolute h-10 left-2 w-28"></div>
        </div>
        <button
          class="w-full px-2 py-1 text-sm font-light text-white bg-indigo-700 rounded-lg disabled:opacity-40 hover:bg-indigo-800"
        >
          <i class="mr-1 text-xs fas fa-file-pdf"></i> PDF
        </button>
        <ul
          class="absolute hidden mt-1 text-white rounded-lg shadow-big w-max dropdown-menu"
        >
          <li class="">
            <a
              @click="getExamPreview(exam, true)"
              class="block px-6 py-4 whitespace-no-wrap bg-indigo-700 rounded-t-lg hover:bg-indigo-800"
              href="#"
              >Scarica esame completo</a
            >
          </li>
          <li class="">
            <a
              @click="getExamPreview(exam, false)"
              class="block px-6 py-4 whitespace-no-wrap bg-indigo-700 rounded-b-lg hover:bg-indigo-800"
              href="#"
              >Scarica una possibile simulazione</a
            >
          </li>
        </ul>
      </div>

      <div v-if="exam.closed" class="inline-block text-left ">
        <div class="inline-block w-full dropdown md:w-max">
          <div class="relative">
            <div class="absolute h-10 left-2 w-28"></div>
          </div>
          <button
            class="w-full px-2 py-1 text-sm font-light text-white align-middle bg-indigo-700 rounded-lg disabled:opacity-40 hover:bg-indigo-800"
          >
            <i class="mr-1 text-xs fas fa-download"></i>

            Risultati
          </button>
          <ul
            class="absolute hidden mt-1 text-white rounded-lg shadow-big w-max dropdown-menu"
          >
            <li class="">
              <a
                @click="getCSVReport(exam)"
                class="block px-6 py-4 whitespace-no-wrap bg-indigo-700 rounded-t-lg hover:bg-indigo-800"
                href="#"
                >Scarica come CSV</a
              >
            </li>
            <li class="">
              <a
                @click="getZipArchive(exam)"
                class="block px-6 py-4 whitespace-no-wrap bg-indigo-700 rounded-b-lg hover:bg-indigo-800"
                href="#"
                >Scarica come PDF</a
              >
            </li>
          </ul>
        </div>
      </div>

      <div class="inline-block w-full dropdown md:w-max">
        <div class="relative">
          <div class="absolute h-10 left-2 w-28"></div>
        </div>
        <button
          class="w-full px-2 py-1 text-sm font-light text-white align-middle bg-indigo-700 rounded-lg disabled:opacity-40 hover:bg-indigo-800"
        >
          <i class="mr-1 text-xs fas fa-download"></i>

          Esporta
        </button>
        <ul
          class="absolute hidden mt-1 text-white rounded-lg shadow-big w-max dropdown-menu"
        >
          <li v-if="exam.has_questions" class="">
            <a
              @click="exportExamItems('q')"
              :class="{ 'rounded-b-lg': !exam.has_exercises }"
              class="block px-6 py-4 whitespace-no-wrap bg-indigo-700 rounded-t-lg hover:bg-indigo-800"
              href="#"
              >Esporta domande</a
            >
          </li>
          <li v-if="exam.has_exercises" class="">
            <a
              @click="exportExamItems('e')"
              :class="{ 'rounded-t-lg': !exam.has_questions }"
              class="block px-6 py-4 whitespace-no-wrap bg-indigo-700 rounded-b-lg hover:bg-indigo-800"
              href="#"
              >Esporta esercizi JS</a
            >
          </li>
        </ul>
      </div>
      <button
        @click="showExamInstructions(exam)"
        v-if="!exam.closed"
        class="px-2 py-1 text-sm font-light text-white align-middle bg-indigo-700 rounded-lg disabled:opacity-40 hover:bg-indigo-800"
      >
        <i class="mr-1 text-xs fas fa-link"></i>
        Cod. accesso
      </button>
      <router-link :to="`/exam/${exam.id}/simulate`"
        ><button
          class="w-full px-2 py-1 text-sm font-light text-white align-middle bg-indigo-700 rounded-lg disabled:opacity-40 hover:bg-indigo-800"
        >
          <i class="mr-1 fas fa-user "></i>

          Simula studente
        </button></router-link
      >
      <router-link
        v-if="exam.closed && !hideStats"
        :to="`/exams/${exam.id}/stats`"
        ><button
          class="w-full px-2 py-1 text-sm font-light text-white align-middle bg-indigo-700 rounded-lg md:w-max disabled:opacity-40 hover:bg-indigo-800"
        >
          <i class="mr-1 fas fa-chart-bar "></i>

          Statistiche
        </button></router-link
      >
    </div>

    <!-- end left buttons -->

    <!--right buttons -->
    <div
      class="flex flex-col items-center space-y-2 md:space-y-0 md:space-x-1 md:ml-auto md:flex-row"
    >
      <div class="px-2 text-sm bg-gray-600 rounded-md" v-if="exam.closed">
        <span class="text-white ">Terminato</span>
      </div>
      <div class="px-2 text-sm bg-red-800 rounded-md" v-if="outOfTimeNotClosed">
        <span class="text-sm text-white">Fuori tempo</span>
      </div>
      <div
        class="px-2 text-sm bg-green-700 rounded-md animate-pulse"
        v-if="startedNotClosed"
      >
        <span class="text-white ">In corso</span>
      </div>
      <div class="px-2 text-sm bg-red-500 rounded-md" v-if="exam.draft">
        <span class="text-white ">Bozza</span>
      </div>
      <div
        class="px-2 bg-gray-700 rounded-md animate-pulse"
        v-if="exam.locked_by"
      >
        <span style="letter-spacing: -0.7px " class="text-white"
          >Modifica in corso da {{ exam.locked_by }}
        </span>
      </div>
      <div class="text-xs text-gray-700 md:pl-1.5">
        <i class="mr-1 text-gray-500 far fa-calendar"></i>
        <span
          v-html="
            formatTimestampShort([exam.begin_timestamp, exam.end_timestamp])
          "
        ></span>
      </div>
    </div>
    <Dialog
      v-if="dialog.shown"
      :string="dialog.string"
      :subText="dialog.subText"
      :confirmOnly="dialog.confirmOnly"
      :dismissible="dialog.dismissible"
      :severity="dialog.severity"
      @yes="dialog.onYes.callback(dialog.onYes.param)"
      @no="dialog.onNo.callback(dialog.onNo.param)"
    ></Dialog>
  </div>
</template>

<script>
import axios from 'axios'
import Spinner from './Spinner.vue'
import Dialog from './Dialog.vue'
import {
  getUserFullName,
  formatTimestampShort,
  getExamSummaryText,
  getExamInstructions,
  truncateString,
  redirectIfPermissionErrorOrSetMessage
} from '../utility'
import {
  forceFileDownload,
  beforeDownload,
  downloadObjectAsJson
} from '../filedownloads'
export default {
  name: 'ExamListItem',
  components: {
    Spinner,
    Dialog
  },
  data () {
    return {
      loading: false,
      loadingMessage: '',
      processedReports: -1,
      totalReports: 0,
      pollIntervalHandle: null,
      pollForResults: false,
      awaitingResponse: false,
      dialog: {
        shown: false
      }
    }
  },
  props: {
    exam: {
      type: Object
    },
    old: {
      type: Boolean,
      default: false
    },
    hideStats: {
      type: Boolean,
      default: false
    }
  },
  watch: {
    pollForResults (newVal) {
      if (newVal) {
        this.pollIntervalHandle = setInterval(() => {
          if (!this.awaitingResponse) {
            // this is used to prevent client from sending multiple requests if the response
            // takes longer to arrive than the time between consecutive requests
            this.getZipArchive()
          }
        }, 2500)
      } else {
        clearInterval(this.pollIntervalHandle)
        this.pollIntervalHandle = null
        this.loadingMessage = ''
      }
    },
    processedReports (newVal) {
      this.loadingMessage = `Generazione dei report in corso. Questo processo potrebbe impiegare alcuni minuti...<br />
          (${newVal} file generati su ${this.totalReports})
          `
    }
  },
  methods: {
    truncateString,
    formatTimestampShort,
    getUserFullName,
    getExamSummaryText,
    getExamInstructions,
    beforeDownload,
    downloadObjectAsJson,
    forceFileDownload,
    confirmClosure (exam) {
      // shows a dialog that prompts the user for confirmation to close an exam
      this.dialog = {
        shown: true,
        string: 'Sei sicuro di voler chiudere questo esame?',
        subText: this.getExamSummaryText(exam),
        confirmOnly: false,
        severity: 2,
        dismissible: true,
        onYes: { callback: this.closeExam, param: exam.id },
        onNo: {
          callback: () => (this.dialog = { shown: false })
        }
      }
    },
    closeExam (id) {
      this.loading = true
      this.dialog = {
        show: false
      }
      axios
        .patch(`/exams/${id}/terminate/`)
        .then(response => {
          // todo refactor this (this changes the prop)
          this.exam = response.data
          this.$store.commit('setSmallMessage', {
            severity: 1,
            msg: 'Esame chiuso con successo.'
          })
        })
        .catch(error => {
          redirectIfPermissionErrorOrSetMessage(
            this,
            error,
            '/login/teacher',
            "Si è verificato un errore chiudendo l'esame. Riprova.",
            false
          )
        })
        .finally(() => {
          this.loading = false
        })
    },
    getExamPreview (exam, allItems) {
      this.loading = true
      axios
        .post(
          `/exams/${exam.id}/${allItems ? 'all_items' : 'mock'}/`,
          {},
          {
            responseType: 'blob'
          }
        )
        .then(response => {
          this.forceFileDownload(
            response,
            `${allItems ? 'Completo' : 'Simulazione'}_${exam.name}.pdf`
          )
        })
        .catch(error => {
          redirectIfPermissionErrorOrSetMessage(
            this,
            error,
            '/login/teacher',
            "Si è verificato un errore scaricando l'esame. Riprova.",
            false
          )
        })
        .finally(() => {
          this.loading = false
        })
    },
    getCSVReport () {
      this.loading = true
      axios
        .post(
          `exams/${this.exam.id}/csv_report/`,
          {},
          {
            responseType: 'blob'
          }
        )
        .then(response => {
          this.forceFileDownload(response, `${this.exam.name}.csv`)
        })
        .catch(error => {
          redirectIfPermissionErrorOrSetMessage(
            this,
            error,
            '/login/teacher',
            'Si è verificato un errore scaricando i risultati. Riprova.',
            false
          )
        })
        .finally(() => {
          this.loading = false
        })
    },
    exportExamItems (itemType) {
      this.loading = true
      axios
        .get(
          `exams/${this.exam.id}/export_${
            itemType == 'q' ? 'questions' : 'exercises'
          }/`
        )
        .then(response => {
          console.log(response.data)
          this.downloadObjectAsJson(
            response.data,
            `${this.exam.name}_${
              itemType == 'q' ? 'domande' : 'eserciziJS'
            }.json`
          )
        })
        .catch(error => {
          redirectIfPermissionErrorOrSetMessage(
            this,
            error,
            '/login/teacher',
            'Si è verificato un errore scaricando i risultati. Riprova.',
            false
          )
        })
        .finally(() => {
          this.loading = false
        })
    },
    async getZipArchive () {
      this.loading = true
      this.awaitingResponse = true
      axios
        .post(
          `exams/${this.exam.id}/zip_archive/`,
          {},
          {
            responseType: 'blob'
          }
        )
        .then(async response => {
          if (response.status == 202) {
            // just scheduled - periodically poll server to see if results are ready
            this.pollForResults = true
            this.processedReports = 0
          } else if (response.status == 206) {
            // still generating reports
            if (!this.pollForResults) {
              // someone else scheduled the generation
              this.pollForResults = true
            }
            const responseDataAsText = JSON.parse(
              await new Response(response.data).text()
            )
            this.processedReports = responseDataAsText.processed
            this.totalReports = responseDataAsText.total
          } else {
            // server finished processing
            this.pollForResults = false
            this.forceFileDownload(response, `${this.exam.name}.zip`)
            this.loading = false
          }
        })
        .catch(error => {
          this.loading = false
          redirectIfPermissionErrorOrSetMessage(
            this,
            error,
            '/login/teacher',
            'Si è verificato un errore scaricando i risultati. Riprova.',
            false
          )
        })
        .finally(() => {
          this.awaitingResponse = false
        })
    },
    showExamInstructions (exam) {
      // shows a dialog that prompts the user for confirmation to close an exam
      this.dialog = {
        shown: true,
        string: "Istruzioni per l'accesso all'esame",
        subText: this.getExamInstructions(exam),
        confirmOnly: true,
        severity: 1,
        dismissible: true,
        onYes: {
          callback: () => (this.dialog = { shown: false })
        }
      }
    }
  },
  computed: {
    notStartedNotClosed () {
      return (
        // the replace is due to the way iPhones work with timestamp
        new Date() < new Date(this.exam.begin_timestamp.replace(' ', 'T')) &&
        !this.exam.closed
      )
    },
    startedNotClosed () {
      return (
        new Date() >= new Date(this.exam.begin_timestamp.replace(' ', 'T')) &&
        !this.exam.closed
      )
    },
    notStartedOrClosed () {
      return (
        new Date() < new Date(this.exam.begin_timestamp.replace(' ', 'T')) ||
        this.exam.closed
      )
    },
    outOfTimeNotClosed () {
      return (
        new Date() >= new Date(this.exam.end_timestamp.replace(' ', 'T')) &&
        !this.exam.closed
      )
    }
  }
}
</script>

<style>
.dropdown:hover .dropdown-menu {
  display: block;
  z-index: 100;
}
.shadow-big {
  box-shadow: 0 0 25px rgba(0, 0, 0, 0.4);
}
</style>
