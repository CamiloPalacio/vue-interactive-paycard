<template>
    <div class="card-form-user">
      <div class="card-list" style="padding-left: 23%">
        <img src="../assets/images/perfil2.png" width="70%">
      </div>
      <div class="card-form-user__inner">
      <div class="card-input">
        <label for="cardName" class="card-input__label">Nombres</label>
        <input
          type="text"
          :id="fields.userName"
          @input="changeName"
          class="card-input__input"
          :value="userData.userName"
          v-letter-only
          autocomplete="off"
          maxlength="30"
          v-required
        />
      </div>
      <div class="card-input">
        <label for="cardNumber" class="card-input__label">Apellidos</label>
        <input
          type="text"
          :id="fields.userLastN"
          @input="changeLastN"
          class="card-input__input"
          :value="userData.userLastN"
          v-letter-only
          maxlength="30"
          autocomplete="off"
          v-required
        />
      </div>
      <div class="card-form__row">
        <div class="card-form__col">
          <div class="card-form__group">
            <label for="userDocType" class="card-input__label">Tipo</label>
            <select
              class="card-input__input -select"
              :id="fields.userDocType"
              v-model="userData.userDocType"
              @change="changeDocType"
              v-required
            >
              <option value disabled selected>Tipo</option>
              <option
                v-bind:value="$index.ID"
                v-for="($index) in documents"
                v-bind:key="$index.ID"
              >{{$index.ABBR}}</option>
            </select>
          </div>
        </div>
        <div class="card-form__col">
          <div class="card-input">
            <label for="cardCvv" class="card-input__label">Número de documento</label>
            <input
              type="tel"
              class="card-input__input"
              v-number-only
              @input="changeDocNumber"
              :id="fields.userDocNum"
              :value="userData.userDocNum"
              autocomplete="off"
              v-required
            />
          </div>
        </div>
      </div>
      <div class="card-input">
        <label for="cardNumber" class="card-input__label">Telefono</label>
        <input
          type="tel"
          class="card-input__input"
          v-number-only
          @input="changePhone"
          :id="fields.userPhone"
          :value="userData.userPhone"
          autocomplete="off"
          v-required
        />
      </div>
      <div class="card-form__row">
          <button class="card-form__button" v-on:click="createUser">Create</button>
          <button class="card-form__button" v-on:click="readUser">Read</button>
      </div>
      <div class="card-form__row">
          <button class="card-form__button" v-on:click="updateUser">Update</button>
          <button class="card-form__button" v-on:click="deleteUser">Delete</button>
      </div>
    </div>
  </div>
</template>
<script>
import Vue from 'vue'
import Toast from 'vue-toastification'
import 'vue-toastification/dist/index.css'
Vue.use(Toast)
const headers = { 'Content-Type': 'application/json' }
export default {
  name: 'UserForm',
  directives: {
    'number-only': {
      bind (el) {
        function checkValue (event) {
          event.target.value = event.target.value.replace(/[^0-9]/g, '')
          if (event.charCode >= 48 && event.charCode <= 57) {
            return true
          }
          event.preventDefault()
        }
        el.addEventListener('keypress', checkValue)
      }
    },
    'letter-only': {
      bind (el) {
        function checkValue (event) {
          if (event.charCode >= 48 && event.charCode <= 57) {
            event.preventDefault()
          }
          return true
        }
        el.addEventListener('keypress', checkValue)
      }
    },
    'required': {
      bind (el) {
        function checkValue (event) {
          if (event.target.value === '') {
            el.style.borderColor = 'red'
          } else {
            el.style.borderColor = null
          }
        }
        el.addEventListener('blur', checkValue)
        el.addEventListener('keyup', checkValue)
        el.addEventListener('focus', checkValue)
        el.addEventListener('change', checkValue)
      }
    }
  },
  props: {
    userData: {
      default: () => {
        return {
          userName: '',
          userLastN: '',
          userDocType: '',
          userDocNum: '',
          userPhone: '',
          userId: null
        }
      }
    },
    initialForm: {
      default: () => {
        return {
          userName: '',
          userLastN: '',
          userDocType: '',
          userDocNum: '',
          userPhone: '',
          userId: null
        }
      }
    }
  },
  data () {
    return {
      fields: {
        userName: 'v-user-name',
        userLastN: 'v-user-last-name',
        userDocType: 'v-user-doc-type',
        userDocNum: 'v-user-doc-number',
        userPhone: 'v-user-phone'
      },
      documents: []

    }
  },
  mounted () {
    const headers = { 'Content-Type': 'application/json' }
    fetch('http://52.200.169.154:8081/documents-type/all', { headers })
      .then(res => res.json())
      .then(({ result }) => { this.documents = result })
      .catch(() =>
        this.$toast('Ha ocurrido un error', {
          timeout: 2000,
          type: 'error',
          position: 'bottom-left'
        }))
  },
  methods: {
    changeName (e) {
      this.userData.userName = e.target.value
      this.$emit('input-user-name', this.userData.userName)
    },
    changeLastN (e) {
      this.userData.userLastN = e.target.value
      this.$emit('input-user-last-name', this.userData.userLastN)
    },
    changeDocType () {
      this.$emit('input-user-doc-type', this.userData.userDocType)
    },
    changeDocNumber (e) {
      this.userData.userDocNum = e.target.value
      this.$emit('input-user-doc-number', this.userData.userDocNum)
    },
    changePhone (e) {
      this.userData.userPhone = e.target.value
      this.$emit('input-user-phone', this.userData.userPhone)
    },
    createUser () {
      fetch('http://52.200.169.154:8081/user/create', {
        headers,
        method: 'POST',
        body: JSON.stringify(this.userData)
      })
        .then(res => res.json())
        .then((data) => {
          const { result, status } = data
          if (status) {
            this.$toast('Usuario Creado', {
              timeout: 2000,
              type: 'success',
              position: 'bottom-left'
            })
            this.userData.userName = ''
            this.userData.userLastN = ''
            this.userData.userDocType = ''
            this.userData.userDocNum = ''
            this.userData.userPhone = ''
          } else {
            this.$toast(`Error: ${result}`, {
              timeout: 2000,
              type: 'error',
              position: 'bottom-left'
            })
          }
        })
        .catch(() => {
          this.$toast(`Error: Ocurrió un problema desconocido`, {
            timeout: 2000,
            type: 'error',
            position: 'bottom-left'
          })
        })
    },
    readUser () {
      const url = `http://52.200.169.154:8081/user/${this.userData.userDocNum}`
      fetch(url, { headers })
        .then(res => res.json())
        .then(data => {
          const { result, status } = data
          if (status) {
            this.userData.userName = result.userName
            this.userData.userLastN = result.userLastN
            this.userData.userDocType = result.userDocType
            this.userData.userPhone = result.userPhone
            this.userData.userId = result.userId
            this.$toast('Usuario Encontrado', {
              timeout: 2000,
              type: 'success',
              position: 'bottom-left'
            })
          } else {
            this.$toast(`Error: ${result}`, {
              timeout: 2000,
              type: 'error',
              position: 'bottom-left'
            })
          }
        })
        .catch(() => {
          this.$toast(`Error: Ocurrió un problema desconocido`, {
            timeout: 2000,
            type: 'error',
            position: 'bottom-left'
          })
        })
    },
    updateUser () {
      const { userId, ...rest } = this.userData
      const url = `http://52.200.169.154:8081/user/update/${userId}`
      console.log(url, rest)
      fetch(url, {
        headers,
        method: 'PATCH',
        body: JSON.stringify(rest)
      })
        .then(res => res.json())
        .then((data) => {
          const { status, message } = data
          if (status) {
            this.$toast('Usuario Modificado', {
              timeout: 2000,
              type: 'success',
              position: 'bottom-left'
            })
            this.userData.userName = ''
            this.userData.userLastN = ''
            this.userData.userDocType = ''
            this.userData.userPhone = ''
          } else {
            this.$toast(`Error: ${message}`, {
              timeout: 2000,
              type: 'error',
              position: 'bottom-left'
            })
          }
        })
        .catch(() => {
          this.$toast(`Error: Ocurrió un problema desconocido`, {
            timeout: 2000,
            type: 'error',
            position: 'bottom-left'
          })
        })
    },
    deleteUser () {
      const num = this.userData.userDocNum
      if (window.confirm(`Estás segudo de que deseas eliminar la tarjeta: \n Numero de tarjeta: ${num}`)) {
        fetch(`http://52.200.169.154:8081/user/delete/${num}`, {
          headers,
          method: 'DELETE'
        })
          .then(res => res.json())
          .then((data) => {
            const { status, message } = data
            if (status) {
              this.$toast('Usuario Eliminado', {
                timeout: 2000,
                type: 'success',
                position: 'bottom-left'
              })
              this.userData.userName = ''
              this.userData.userLastN = ''
              this.userData.userDocType = ''
              this.userData.userPhone = ''
            } else {
              this.$toast(`Error: ${message}`, {
                timeout: 2000,
                type: 'error',
                position: 'bottom-left'
              })
            }
          })
          .catch(() => {
            this.$toast(`Error: Ocurrió un problema desconocido`, {
              timeout: 2000,
              type: 'error',
              position: 'bottom-left'
            })
          })
      }
    }
  }
}
</script>
