<template>
  <v-layout wrap class="white-background pa-5">
    <v-flex xs12 sm12 md4>
      <v-text-field
        placeholder="Search"
        hide-details
        prepend-inner-icon="mdi-magnify"
        @keydown="searchUser"
      ></v-text-field>
    </v-flex>
    <v-flex xs12 sm12 md12>
      <users-table
        :userList="users"
        :loading="loading"
        @onDelete="onDelete($event)"
        @onApprove="onApprove($event)"
        @onReject="onReject($event)"
      ></users-table>
    </v-flex>
    <app-snackbar :snackbar="snackbar" @snackbarAction="snackbarAction($event)"></app-snackbar>

    <v-menu
      :close-on-content-click="false"
      :close-on-click="false"
      v-model="rejectCommentMenu"
      content-class="userrequests--rejectComment"
      activator=".user-actions"
      offset-y
      min-width="300px"
    >
      <v-card light>
        <v-card-text>
          <v-textarea label="User Rejection Comment" v-model="rejectionComment"></v-textarea>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
            text
            class="caption"
            color="red"
            :disabled="!rejectionComment"
            @click="rejectConfirmation"
          >Reject</v-btn>
          <v-btn text class="caption" @click="rejectCommentMenu = false">Cancel</v-btn>
        </v-card-actions>
      </v-card>
    </v-menu>
  </v-layout>
</template>

<script>
import UsersTable from '~/components/UsersTable'

export default {
  layout: 'main',
  components: { UsersTable },
  middleware: 'admin',

  data: () => ({
    title: 'Users Requests',
    valid: false,
    users: [],
    unfilteredUsers: [],
    loading: false,
    snackbar: {},
    selectedUser: {},
    actionType: '',
    rejectCommentMenu: false,
    rejectionComment: ''
  }),

  computed: {
    rules() {
      return this.$store.state.app.inputRules
    }
  },

  head() {
    return {
      titleTemplate: `${this.title} - %s`
    }
  },

  mounted() {
    this.$store.dispatch('app/setAppTitle', this.title)
    this.getPendingUsers()
  },

  methods: {
    getPendingUsers() {
      this.loading = true
      axios
        .get(`${process.env.adminUrl}Accounts/User/GetPending`)
        .then(response => {
          this.loading = false
          if (response && response.data.success) {
            this.users = response.data.result
            this.unfilteredUsers = response.data.result
          }
        })
    },

    searchUser(e) {
      if (e.target.value) {
        this.users = this.users.filter(
          user =>
            user.fullName.includes(e.target.value) ||
            user.email.includes(e.target.value)
        )
      } else this.users = this.unfilteredUsers
    },

    onDelete(user) {
      this.selectedUser = user
      this.actionType = 'delete'
      this.snackbar = {
        show: true,
        text: `Are you sure you want to delete ${user.fullName}?`,
        timeout: 0,
        actions: ['Yes', 'No']
      }
    },

    deleteUser() {
      this.loading = true
      this.snackbar.show = false
      axios
        .post(
          `${process.env.adminUrl}accounts/user/${this.selectedUser.id}/delete`
        )
        .then(response => {
          if (response && response.data.success) {
            this.users = this.users.filter(
              user => user.id != this.selectedUser.id
            )

            window.getApp.snackbar = {
              show: true,
              text: response.data.message
            }
          }
        })
        .finally(() => {
          this.loading = false
        })
    },

    onApprove(user) {
      this.selectedUser = user
      this.actionType = 'approve'
      this.snackbar = {
        show: true,
        text: `Are you sure you want to approve ${user.fullName}?`,
        timeout: 0,
        actions: ['Yes', 'No']
      }
    },

    approveUser() {
      this.loading = true
      this.snackbar.show = false
      axios
        .post(
          `${process.env.adminUrl}accounts/user/${this.selectedUser.id}/approved`
        )
        .then(response => {
          if (response && response.data.success) {
            this.users = this.users.filter(
              user => user.id != this.selectedUser.id
            )

            window.getApp.snackbar = {
              show: true,
              text: response.data.message
            }
          }
        })
        .finally(() => {
          this.loading = false
        })
    },

    onReject(user) {
      this.selectedUser = user
      this.rejectCommentMenu = true
    },

    rejectConfirmation() {
      this.actionType = 'reject'
      this.snackbar = {
        show: true,
        text: `Are you sure you want to reject ${this.selectedUser.fullName}?`,
        timeout: 0,
        actions: ['Yes', 'No']
      }
    },

    rejectUser() {
      this.loading = true
      this.rejectCommentMenu = false
      this.snackbar.show = false
      axios
        .post(
          `${process.env.adminUrl}accounts/user/rejected?id=${this.selectedUser.id}&rejectionComment=${this.rejectionComment}`
        )
        .then(response => {
          if (response && response.data.success) {
            this.users = this.users.filter(
              user => user.id != this.selectedUser.id
            )

            window.getApp.snackbar = {
              show: true,
              text: response.data.message
            }
          }
        })
        .finally(() => {
          this.loading = false
        })
    },

    snackbarAction(action) {
      switch (this.actionType) {
        case 'delete':
          switch (action) {
            case 'Yes':
              this.deleteUser()
              break
            case 'No':
              this.snackbar.show = false
              break
          }
          break
        case 'approve':
          switch (action) {
            case 'Yes':
              this.approveUser()
              break
            case 'No':
              this.snackbar.show = false
              break
          }
          break
        case 'reject':
          switch (action) {
            case 'Yes':
              this.rejectUser()
              break
            case 'No':
              this.snackbar.show = false
              this.rejectCommentMenu = false
              this.rejectionComment = ''
              break
          }
          break
      }
    }
  }
}
</script>
