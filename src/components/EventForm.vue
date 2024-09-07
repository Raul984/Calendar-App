<template>
  <div class="modal">
    <div class="modal-content">
      <span @click="$emit('cancel')" class="close">&times;</span>
      <h3>{{ isEditMode ? 'Edit event' : 'Add event' }}</h3>
      <form @submit.prevent="submitForm">
        <div class="form-group">
          <label for="title">Event name</label>
          <input type="text" id="title" v-model="formEvent.title" :maxlength="30" placeholder="Enter event title (max 30 characters)" required />
        </div>

        <div class="form-group">
          <label for="timeStart">Event start</label>
          <input type="time" id="timeStart" v-model="formEvent.timeStart" required />
        </div>

        <div class="form-group">
          <label for="timeEnd">Event end</label>
          <input type="time" id="timeEnd" v-model="formEvent.timeEnd" @blur="validateTime" required />
        </div>

        <div class="form-group"> 
          <label for="color">Color</label>
          <input type="color" id="color" v-model="formEvent.color" />
        </div>

        <div class="form-group">
          <label for="notes">Notes</label>
          <textarea id="notes" v-model="formEvent.notes"></textarea>
        </div>

        <div class="form-group buttons">
          <button type="button" @click="$emit('cancel')" class="cancel-button">Cancel</button>
          <button type="submit" class="save-button">Save</button>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    event: {
      type: Object,
      default: () => ({ title: '', date: '', timeStart: '', timeEnd: '', color: '', notes: '', class: '' })
    },
    isEditMode: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      formEvent: { ...this.event } // Копия объекта event для редактирования
    };
  },
  watch: {
    event: {
      deep: true,
      handler(newEvent) {
        this.formEvent = { ...newEvent }; // Обновляем форму при изменении переданного события
      }
    }
  },
  methods: {
    submitForm() {
      // Отправка данных формы через emit события 'submit'
      this.$emit('submit', { ...this.formEvent });
    },
    validateTime() {
      // Валидация времени
      const [startHour, startMinute] = this.formEvent.timeStart.split(':').map(Number);
      const [endHour, endMinute] = this.formEvent.timeEnd.split(':').map(Number);

      const startTotalMinutes = startHour * 60 + startMinute;
      const endTotalMinutes = endHour * 60 + endMinute;

      if (endTotalMinutes <= startTotalMinutes) {
        alert('The end time cannot be less than or equal to the start time.');
        this.formEvent.timeEnd = ''; // Сбросить неверное значение
      }
    }
  }
};
</script>

<style scoped>
.modal {
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  z-index: 2;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.4);
}

.modal-content {
  background-color: white;
  padding: 20px;
  border: 1px solid #888;
  width: 400px;
  border-radius: 10px;
}

.close {
  color: #aaa;
  float: right;
  font-size: 28px;
  font-weight: bold;
  cursor: pointer;
}

.close:hover,
.close:focus {
  color: black;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: flex;
  margin-bottom: 5px;
}

.form-group input,
.form-group textarea {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  box-sizing: border-box;
  min-height: 40px;
}

.buttons {
  display: flex;
  justify-content: space-between;
}

.save-button {
  background-color: #007bff;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.save-button:hover {
  background-color: #0056b3;
}

.cancel-button {
  background-color: transparent;
  color: #ff0000;
  padding: 10px 20px;
  border: none;
  cursor: pointer;
}

.cancel-button:hover {
  color: #b30000;
}
</style>