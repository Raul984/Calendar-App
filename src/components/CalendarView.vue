<template>
  <div>
    <vue-cal
      ref="calendar"
      style="height: 800px;"
      :events="events"
      draggable
      :active-view="currentView"
      events-on-month-view="short"
      events-count-on-year-view
      :time="true"
      editable-events
      @event-click="editEvent"
      @cell-click="addEvent"
      @event-drop="onEventDrop"
      @event-duration-change="onEventDrop"
      @event-delete="onEventDelete"
      @view-change="onViewChange"
      class="vuecal--blue-theme"
    />
      
    <EventForm
      v-if="showForm"
      :event="selectedEvent"
      :isEditMode="isEditMode"
      @submit="saveEvent"
      @cancel="cancelForm"
    />
  </div>
</template>

<script>
import VueCal from 'vue-cal';
import EventForm from './EventForm.vue';

export default {
  components: {
    VueCal,
    EventForm
  },
  data() {
    return {
      events: [], // Список событий
      showForm: false, // Отображение формы
      selectedEvent: null, // Выбранное событие
      isEditMode: false, // Режим редактирования
      currentView: 'month' // Текущий вид календаря (по умолчанию месяц)
    };
  },
  watch: {
    events: {
      deep: true,
      handler(newEvents) {
        this.saveEventsToLocalStorage(newEvents); // Сохранение изменений в localStorage
      }
    }
  },
  mounted() {
    this.loadEventsFromLocalStorage(); // Загрузка событий при создании компонента
    this.$nextTick(() => {
      this.applyAllEventColors(); // Применяем цвета после загрузки
    });
  },
  methods: {
    // Метод для загрузки событий из localStorage
    loadEventsFromLocalStorage() {
      const savedEvents = localStorage.getItem('events');
      if (savedEvents) {
        this.events = JSON.parse(savedEvents).map(event => ({
          ...event,
          start: new Date(event.start),
          end: new Date(event.end)
        }));
      }
    },

    // Метод для сохранения событий в localStorage
    saveEventsToLocalStorage(events) {
      const eventsToStore = events.map(event => ({
        ...event,
        start: event.start.toISOString(),
        end: event.end.toISOString()
      }));
      localStorage.setItem('events', JSON.stringify(eventsToStore));
      this.$nextTick(() => {
        this.applyAllEventColors(); // Применяем цвета после загрузки
      });
    },
    // Метод для добавления нового события
    addEvent(date) {
      if (!(date instanceof Date) || isNaN(date.getTime())) {
        console.error('Invalid date received in addEvent');
        return;
      }

      // Получаем дату в формате YYYY-MM-DD
      const formattedDate = date.toISOString().split('T')[0];
      
      // Получаем время в формате HH:MM
      const timeStart = date.toTimeString().slice(0, 5);
      const timeEnd = '';

      // Инициализируем новое событие с выбранной датой и временем
      this.selectedEvent = { 
        title: '', 
        date: formattedDate, 
        timeStart, 
        timeEnd, 
        notes: '', 
        color: '#3B86FF', 
        class: `event-${new Date(date).getTime()}` 
      };
      this.isEditMode = false;
      this.showForm = true;
    },

    // Метод для редактирования события
    editEvent(event, e) {
      const startDate = new Date(event.start);
      const endDate = new Date(event.end);

      this.selectedEvent = {
        ...event,
        date: startDate.toISOString().split('T')[0],
        timeStart: startDate.toTimeString().slice(0, 5),
        timeEnd: endDate.toTimeString().slice(0, 5)
      };

      this.isEditMode = true;
      this.showForm = true;
    },

    // Метод для сохранения события (добавления или редактирования)
    saveEvent(event) {
      // Преобразуем дату и время в формат для vue-cal
      const startDateTime = `${event.date} ${event.timeStart}`;
      const endDateTime = `${event.date} ${event.timeEnd}`; 

      if (event.isTrusted) {
        return; // Игнорируем встроенные события
      }

      if (this.isEditMode) {
        // Редактирование события
        const index = this.events.findIndex(e => e.id === this.selectedEvent.id);
        if (index !== -1) {
          // Обновляем событие с использованием новых данных
          this.events.splice(index, 1, { ...event, start: new Date(startDateTime), end: new Date(endDateTime) });
        }
      } else {
        // Добавление нового события
        event.id = Date.now(); // Уникальный ID для события
        this.events.push({ ...event, start: new Date(startDateTime), end: new Date(endDateTime) });
      }
      this.showForm = false; // Закрываем форму
    },

    // Метод для отмены формы
    cancelForm() {
      this.showForm = false; // Закрываем форму
    },

    // Применение цвета к DOM элементу по уникальному классу
    applyEventColor(event) {
      const eventElement = this.$refs.calendar.$el.querySelector(`.${event.class}`);
      if (eventElement) {
        eventElement.style.backgroundColor = event.color;
      }
    },

    // Применение цвета ко всем событиям
    applyAllEventColors() {
      this.events.forEach(event => {
        this.applyEventColor(event);
      });
    },

    // Обработка смены вида календаря
    onViewChange() {
      setTimeout(() => {
        this.applyAllEventColors(); // Применяем цвета после изменения вида
      }, 500);
    },

    // Метод для обработки перетаскивания событий в календаре
    onEventDrop({ event }) {
      // Проверяем встроенное событие
      if (event.isTrusted) {
        return; // Игнорируем встроенные события
      }
      // Поиск события по его уникальному ID
      const eventIndex = this.events.findIndex(e => e.id === event.id);
      if (eventIndex !== -1) {
        // Обновляем даты начала и конца события
        this.events[eventIndex].date = event.start.toISOString().split('T')[0];
        this.events[eventIndex].timeStart = event.start.toTimeString().slice(0, 5);
        this.events[eventIndex].timeEnd = event.end.toTimeString().slice(0, 5);
        this.events[eventIndex].start = event.start;
        this.events[eventIndex].end = event.end;
      }
    },

    // Метод для обработки удаления событий в календаре
    onEventDelete(event) {
      // Найдем индекс события
      const eventIndex = this.events.findIndex(e => e.id === event.id);
      if (eventIndex !== -1) {
        // Удаляем событие из массива events
        this.events.splice(eventIndex, 1);
      }
    }
  }
};
</script>

<style scoped>
button {
  margin-right: 10px;
}
</style>
