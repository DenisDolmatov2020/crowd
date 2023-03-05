<template>
  <v-app>
    <v-app-bar
      fixed
      app
    >
      <v-dialog
        v-model="dialog"
        fullscreen
        hide-overlay
        transition="dialog-bottom-transition"
      >
        <template #activator="{ on, attrs }">
          <v-btn
            color="primary"
            dark
            v-bind="attrs"
            v-on="on"
          >
            Проекты
          </v-btn>
        </template>
        <v-app-bar app fixed>
          <v-btn
            icon
            dark
            @click="dialog = false"
          >
            <v-icon>mdi-close</v-icon>
          </v-btn>
          <v-toolbar-title>Проекты</v-toolbar-title>
          <v-spacer />

          <v-row class="align-center justify-center mr-3">
            <v-text-field
              v-model="project.name"
              label="Название проекта"
              outlined
              dense
              class="mt-5 mr-3"
            />

            <v-card :color="project.color" style="width: 30px; height: 30px" />
            <v-select
              v-model="project.color"
              label="Цвет"
              :items="['red', 'green', 'yellow', 'pink', 'blue', 'black', 'indigo', 'purple', 'cyan', 'teal', 'light-blue']"
              outlined
              dark
              dense
              class="mt-5 mx-3"
            />

            <v-btn color="orange" outlined @click="addProject">
              Добавить проект
            </v-btn>
          </v-row>
        </v-app-bar>
        <v-card>
          <v-list class="mt-12">
            <v-list-item
              v-for="p in Object.keys(projects)"
              :key="p"
              :class="projects[p].color"
              class="my-5"
            >
              <v-list-item-content>
                <v-list-item-title class="text--white">
                  Название проекта: {{ p }}
                </v-list-item-title>
                <v-list-item-subtitle>
                  Возможные цены: {{ projects[p].prices }}
                </v-list-item-subtitle>
              </v-list-item-content>

              <v-list-item-action>
                <v-spacer />
                <v-btn outlined color="error" @click="deleteProject(p)">
                  Удалить проект
                </v-btn>
              </v-list-item-action>
            </v-list-item>
            <v-list-item>
              <v-list-item-content>
                <v-list-item-title>{{ price }}</v-list-item-title>
                <v-list-item-subtitle>Require password for purchase or use password to restrict purchase</v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
          </v-list>
        </v-card>

        <v-footer fixed>
          <v-row class="justify-space-around">
            <v-select
              v-model="price.projectName"
              label="Выберете проекта"
              :items="Object.keys(projects)"
              style="max-width: 200px"
            />

            <v-text-field
              v-model="price.value"
              label="Стоимость"
              style="max-width: 200px"
            />

            <v-btn color="orange" class="mt-5" @click="addPrice">
              Добавить цену
            </v-btn>
          </v-row>
        </v-footer>
      </v-dialog>

      <v-spacer />

      <v-row class="mr-3 pt-8">

        <v-col>
          <v-menu
            ref="menu"
            v-model="menu"
            :close-on-content-click="false"
            :return-value.sync="dates"
            transition="scale-transition"
            offset-y
            min-width="auto"
          >
            <template v-slot:activator="{ on, attrs }">
              <v-combobox
                v-model="dates"
                multiple
                chips
                small-chips
                label="Выбор дат"
                prepend-icon="mdi-calendar"
                readonly
                v-bind="attrs"
                v-on="on"
              />
            </template>

            <v-date-picker
              v-model="dates"
              range
              no-title
              :weekday-format="getDay"
              locale="ru-ru"
              clearable
            >
              <v-spacer></v-spacer>
              <v-btn
                text
                color="primary"
                @click="menu = false"
              >
                Cancel
              </v-btn>
              <v-btn
                text
                color="primary"
                @click="$refs.menu.save(dates)"
              >
                OK
              </v-btn>
            </v-date-picker>
          </v-menu>
        </v-col>

        <v-col cols>
          <v-text-field v-model="accountName" label="Название" />
        </v-col>
      </v-row>

      <v-btn color="secondary" @click="fetchData">
        Поиск
      </v-btn>
    </v-app-bar>
    <v-main>
      <v-container class="mt-6">
        <v-card>
          <v-chip
            v-for="(val, proj) in projectsSummary"
            :key="proj"
            :color="val.color"
            x-large
            class="mx-3"
          >
            {{ proj }}: {{ val.summary }} N
          </v-chip>
        </v-card>

        <v-card>
          <v-card-title>
            Транзакции
            <v-spacer />
            <v-text-field
              v-model="search"
              append-icon="mdi-magnify"
              label="Search"
              single-line
              hide-details
            />
          </v-card-title>
          <v-data-table
            :headers="headers"
            :items="items"
            :search="search"
            :items-per-page="5"
            :loading="loading"
          >
            <template #[`item.block_timestamp`]="{ item }">
              <v-chip :color="getColor(item.args.deposit)">
                {{ new Date(parseInt(item.block_timestamp) / 1000000).toLocaleDateString() }}
                {{ new Date(parseInt(item.block_timestamp) / 1000000).toLocaleTimeString() }}
              </v-chip>

            </template>

            <template #[`item.args.deposit`]="{ item }">
              <v-chip :color="getColor(item.args.deposit)">
                {{ (item.args.deposit / 1000000000000000000000000).toFixed(4) }} N
              </v-chip>

            </template>
          </v-data-table>
        </v-card>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
const daysOfWeek = ['Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб', 'Вс']

export default {
  name: 'DefaultLayout',
  data () {
    return {
      dialog: false,
      loading: false,

      dates: ['2023-03-01', '2023-03-04'],
      menu: false,

      accountName: 'napoli_pizzas.crowdforces.near',

      project: {
        name: '',
        color: ''
      },

      price: {
        projectName: '',
        value: ''
      },

      projects: {},

      search: '',
      headers: [
        {
          text: 'Дата',
          value: 'block_timestamp',
          align: 'start'
        },

        {
          text: 'Отправитель',
          value: 'signer_id'
        },

        {
          text: 'Получатель',
          value: 'receiver_id'
        },

        {
          text: 'Сумма',
          value: 'args.deposit',
          align: 'start'
        }
      ],

      items: []
    }
  },

  computed: {
    projectsSummary () {
      const projects = { 'Не определено': { summary: 0 }, Всего: { summary: 0, color: 'blue-grey-darken-1' } }

      for (const p in this.projects) {
        projects[p] = { summary: 0, color: this.projects[p].color }
      }

      for (const item of this.items) {
        let have = false
        for (const project in this.projects) {
          if (this.projects[project].prices.includes(String(item.args.deposit / 1000000000000000000000000))) {
            projects.prices.push(this.projects[project].color)
            projects[project].summary += item.args.deposit / 1000000000000000000000000
            have = true
          }
        }
        if (!have) {
          projects['Не определено'].summary += item.args.deposit / 1000000000000000000000000
        }

        projects['Всего'].summary += item.args.deposit / 1000000000000000000000000
      }

      return projects
    }

  },

  mounted () {
    this.projects = JSON.parse(localStorage.getItem('projects'))
  },

  methods: {
    async fetchData () {
      this.loading = true
      const url = `https://api.kitwallet.app/account/${this.accountName}/activity?limit=1000`
      const { data } = await this.$axios.get(url)

      const startTimestamp = new Date(this.dates[0]).getTime()
      const endTimestamp = new Date(this.dates[1]).getTime() + 86400000 // Add 23:59:59 to end date

      this.items = data.filter((item) => {
        const itemTimestamp = parseInt(item.block_timestamp) / 1000000
        return item.signer_id !== this.accountName &&
          itemTimestamp >= startTimestamp &&
          itemTimestamp <= endTimestamp
      })

      this.loading = false
    },

    getColor (deposit) {
      for (const project of Object.keys(this.projects)) {
        if (this.projects[project].prices.includes(String(deposit / 1000000000000000000000000))) {
          return this.projects[project].color
        }
      }
    },

    getDay (date) {
      const i = new Date(date).getDay(date)
      return daysOfWeek[i]
    },

    addProject () {
      this.projects[this.project.name] = { color: this.project.color, prices: [] }

      localStorage.setItem('projects', JSON.stringify(this.projects))

      this.project = {
        name: '',
        color: ''
      }
    },

    deleteProject (name) {
      delete this.projects[name]
      localStorage.setItem('projects', JSON.stringify(this.projects))

      this.projects = JSON.parse(localStorage.getItem('projects'))
    },

    addPrice () {
      this.projects[this.price.projectName].prices.push(this.price.value)

      localStorage.setItem('projects', JSON.stringify(this.projects))

      this.projects = JSON.parse(localStorage.getItem('projects'))

      this.price.value = ''
    }
  }
}
</script>
