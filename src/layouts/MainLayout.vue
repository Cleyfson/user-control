<template>
  <q-layout class="">

    <q-banner class="q-mb-xl bg-primary text-white">
      <template v-slot:action>
        <q-btn flat color="white" label="Logout" @click="logout()" />
        <q-btn flat color="white" label="Cria novo usuario" @click="criarUsuario()" v-show="role === 3"/>
      </template>
    </q-banner>

    <q-dialog v-model="opened">
      <form-user :showModal="showModal" :newForm="this.usuario" @altereUsuario="altereUsuario">
      </form-user>
    </q-dialog>

    <user-card :usuarios="usuarios" :showModal="showModal" :deletar="deletar" @data="getData" />

    <div class="flex flex-center q-mt-xl q-gutter-xl">
      <q-btn @click="voltar(this.page)" color="primary" icon="navigate_before" />
      <q-btn @click="seguir(this.page, this.totalPages)" color="primary" icon="navigate_next" />
    </div>

  </q-layout>
</template>

<script>
import FormUser from 'src/components/FormUser.vue'
import UserCard from 'src/components/UserCard.vue'
import { mapActions, mapState } from 'vuex'

export default {
  name: 'MainLayout',

  data () {
    return {
      usuarios: null,
      usuariosPorPagina: null,
      pagina: null,
      totalPages: null,
      total: null,
      usuario: {
        id: null,
        nome: null,
        email: null,
        password: null
      },
      opened: null
    }
  },

  computed: {
    ...mapState('usuarios', { role: state => state.usuario.role })
  },

  components: {
    UserCard,
    FormUser
  },

  methods: {
    ...mapActions('usuarios', ['carregaPagina', 'deletarUsuario', 'logout']),

    voltar (pagina) {
      const previousPage = (pagina > 1) ? (pagina - 1) : pagina
      this.$router.push({ path: `/${previousPage}` })
      this.getUsuarios(previousPage)
    },
    seguir (pagina, total) {
      const nextPage = (pagina < total) ? (pagina + 1) : pagina
      this.$router.push({ path: `/${nextPage}` })
      this.getUsuarios(nextPage)
    },
    resetForm () {
      this.id = null
      this.nome = null
    },
    criarUsuario () {
      this.resetForm()
      this.showModal()
    },
    showModal: function () {
      if (this.opened === true) {
        (this.opened = false)
      } else {
        (this.opened = true)
      }
    },
    getData (data) {
      this.usuario.id = data.id_hash
      this.usuario.nome = `${data.first_name} ${data.last_name}`
      this.usuario.email = data.email
      this.usuario.password = data.password
      this.showModal()
    },
    altereUsuario (data) {
      const usuario = this.usuarios.find(usuario => usuario.id_hash === data.id)
      const nameArray = data.nome.split(' ')
      usuario.first_name = nameArray[0]
      usuario.last_name = nameArray[1]
    },
    getUsuarios (pagina) {
      this.carregaPagina(pagina)
        .then((response) => {
          this.usuarios = response.data.data
          this.page = response.data.meta.current_page
          this.perPage = response.data.meta.per_page
          this.total = response.data.meta.total
          this.totalPages = response.data.meta.last_page
        })
    },
    deletar (id) {
      this.deletarUsuario(id)
        .then((response) => {
          this.usuarios = this.usuarios.filter((usuario) => usuario.id_hash !== id)
        })
    }
  },

  mounted () {
    const pagina = this.$route.params.page
    this.getUsuarios(pagina)
  }

}
</script>

<style lang="sass" scoped>
</style>
