<template>
  <div class="wrapper">
    <nav class="main-header navbar navbar-expand navbar-white navbar-light">
      <ul class="navbar-nav">
        <li class="nav-item">
          <a class="nav-link" data-widget="pushmenu" href="#" role="button"><i class="fas fa-bars"></i></a>
        </li>
      </ul>
    </nav>
    <aside class="main-sidebar sidebar-dark-primary elevation-4">
      <h2 class="brand-text font-weight-light mt-2">Task Sprint</h2>
      <div class="sidebar">
        <div class="user-panel pb-3 mb-3 d-flex flex-column align-items-center">
          <h5 class="info">{{ user_name }}</h5>
          <div class="info">{{ user_email }}</div>
          <div class="info">{{ group_name }}</div>
        </div>
      </div>
    </aside>
    <div class="content-wrapper">
      <section class="content-header">
        <div class="container-fluid">
          <div class="row mb-2">
            <div class="col-sm-6">
              <h1>Календарь</h1>
            </div>
          </div>
        </div>
      </section>
      <section class="content">
        <div class="container-fluid">
          <div class="row">
              <div class="card card-primary">
                <div class="card-body p-0">
                  <div id="calendar"></div>
                </div>
              </div>
          </div>
        </div>
      </section>
    </div>
  </div>
  <Popup v-if="popupTrigger.buttonTrigger"
         :TogglePopup ="() => TogglePopup('buttonTrigger')">
    <h2>{{ popupTrigger.title}}</h2>
    <p>{{ popupTrigger.description }}</p>
    <button class="btn btn-primary" @click="TaskDone(popupTrigger.id)">Выполнено</button>
  </Popup>
</template>

<script>
  import {Cookie} from "@/assets/Cookie.js";
  import Popup from "@/components/Popup.vue";
  import {ref} from "vue";
  import axios from "axios";

  const user_name = Cookie.getCookie('user_name')
  const user_email = Cookie.getCookie('user_email')
  const group_name = Cookie.getCookie('group_name')

  export default {
    components: {Popup},
    setup (){
      const popupTrigger = ref({
        buttonTrigger: false,
        title: '',
        description: '',
        id: ''
      })

      function TogglePopup (trigger, title, description, id) {
        popupTrigger.value[trigger] = !popupTrigger.value[trigger]
        popupTrigger.value.title = title
        popupTrigger.value.description = description
        popupTrigger.value.id = id
      }

      function TaskDone(id) {
        axios.post('http://localhost/api/tasks/done/' + id)
            .then(() => {
              location.reload()
            })
      }

      return {
        Popup,
        popupTrigger,
        TogglePopup,
        TaskDone,
      }
    },
    data() {
      return {
        user_name: 'Test Testovich Name',
        user_email: 'example@mail.ru',
        group_name: 'Group_name',
      }
    },
    mounted() {
      this.getTasks(this.$route.params.group_id, this.TogglePopup)
    },
    methods: {
      getTasks(url, popup) {
        this.axios.get('http://localhost/api/tasks/' + url)
            .then(res => {
              $(function () {

                let Calendar = FullCalendar.Calendar;
                let calendarEl = document.getElementById('calendar');
                let calendar = new Calendar(calendarEl, {
                  locale: 'ru',
                  headerToolbar: {
                    left  : 'prev,next today',
                    center: 'title',
                    right : ''
                  },
                  themeSystem: 'bootstrap',
                  events: res.data.data,
                  eventClick: function(info) {
                    console.log(info.event._def.title);
                    popup('buttonTrigger', info.event._def.title, info.event._def.extendedProps.description, info.event._def.publicId)
                  },
                });

                calendar.render();
              })
            })
      }
    },
    name: "UserDesktop",
  }


</script>

<style scoped>
.brand-text {
  color: white;
  text-align: center;
}
.info {
  color: white;
}
</style>