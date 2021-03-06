<template>

  <div class="container-fluid">

    <!-- page error -->
    <div v-if="error"
      class="alert alert-danger">
      <span class="fa fa-exclamation-triangle">
      </span>&nbsp;
      {{ error }}
      <button type="button"
        class="close cursor-pointer"
        @click="error = ''">
        <span>&times;</span>
      </button>
    </div> <!-- /page error -->

    <!-- page success -->
    <div v-if="success"
      class="alert alert-success">
      <span class="fa fa-check">
      </span>&nbsp;
      {{ success }}
      <button type="button"
        class="close cursor-pointer"
        @click="success = ''">
        <span>&times;</span>
      </button>
    </div> <!-- /page success -->

    <!-- password set but user is not logged in -->
    <div v-if="!error && hasAuth && !loggedIn"
      class="alert alert-danger">
      <span class="fa fa-exclamation-triangle">
      </span>&nbsp;
      This page requires admin privileges. Please login.
    </div> <!-- /password set but user is not logged in -->

    <!-- page content -->
    <div class="row">

      <!-- navigation -->
      <div v-if="hasAuth && loggedIn"
        class="col-xl-2 col-lg-3 col-md-3 col-sm-4"
        role="tablist"
        aria-orientation="vertical">
        <div class="nav flex-column nav-pills">
          <a class="nav-link cursor-pointer"
            @click="openView('general')"
            :class="{'active':visibleTab === 'general'}">
            <span class="fa fa-fw fa-cog">
            </span>&nbsp;
            General
          </a>
          <a class="nav-link cursor-pointer"
            @click="openView('password')"
            :class="{'active':visibleTab === 'password'}">
            <span class="fa fa-fw fa-lock">
            </span>&nbsp;
            Password
          </a>
          <a class="nav-link cursor-pointer"
            @click="openView('notifiers')"
            :class="{'active':visibleTab === 'notifiers'}">
            <span class="fa fa-fw fa-bell">
            </span>&nbsp;
            Notifiers
          </a>
        </div>
      </div> <!-- /navigation -->

      <!-- general -->
      <div v-if="visibleTab === 'general' && hasAuth && loggedIn"
        class="col">
        <h3>
          General
        </h3>
        <hr>
        <div class="row">
          <div class="col-lg-9 col-md-12 form-group">
            <label for="outOfDate">
              Seconds before an Elasticsearch node is considered out of date
            </label>
            <input type="number"
              class="form-control"
              id="outOfDate"
              @input="debounceInput"
              v-model="settings.general.outOfDate"
            />
            <p class="form-text small text-muted">
              Will add an
              <strong>Out Of Date</strong>
              issue to the node's cluster if the node has not checked in within the specified number of seconds.
            </p>
          </div>
          <div class="col-lg-9 col-md-12 form-group">
            <label for="esQueryTimeout">
              Seconds until Elasticsearch queries to get cluster health and stats time out
            </label>
            <input type="number"
              class="form-control"
              id="esQueryTimeout"
              @input="debounceInput"
              v-model="settings.general.esQueryTimeout"
            />
            <p class="form-text small text-muted">
              Aborts the queries and adds an
              <strong>ES Down</strong>
              issue if no response is received within the specified time.
            </p>
          </div>
        </div>
      </div>
      <!-- /general -->

      <!-- password -->
      <div v-if="(visibleTab === 'password' && hasAuth && loggedIn) || (visibleTab === 'password' && !hasAuth)"
        class="col">
        <h3 class="mb-3">
          Password
          <span v-if="passwordChanged"
            class="pull-right">
            <!-- cancel password update button -->
            <a @click="cancelChangePassword()"
              class="btn btn-outline-warning cursor-pointer">
              <span class="fa fa-ban">
              </span>&nbsp;
              Cancel
            </a> <!-- /cancel password update button -->
            <!-- update/create password button -->
            <a v-if="(hasAuth && loggedIn) || !hasAuth"
              @click="updatePassword()"
              class="btn btn-outline-success cursor-pointer mr-1 ml-1">
              <span class="fa fa-key"></span>
              <span v-if="hasAuth && loggedIn">
                Update
              </span>
              <span v-if="!hasAuth">
                Create
              </span>
              Password
            </a> <!-- /update/create password button -->
          </span>
        </h3>
        <hr>
        <div v-if="hasAuth && loggedIn"
          class="input-group mb-2">
          <span class="input-group-prepend">
            <span class="input-group-text">
              Current Password
            </span>
          </span>
          <input class="form-control"
            @keyup.enter="updatePassword()"
            name="currentPassword"
            @input="passwordChanged = true"
            v-model="currentPassword"
            type="password"
          />
        </div>
        <div class="input-group mb-2">
          <span class="input-group-prepend">
            <span class="input-group-text">
              New Password
            </span>
          </span>
          <input class="form-control"
            name="newPassword"
            @keyup.enter="updatePassword()"
            @input="passwordChanged = true"
            v-model="newPassword"
            type="password"
          />
        </div>
        <div class="input-group mb-2">
          <span class="input-group-prepend">
            <span class="input-group-text">
              Confirm New Password
            </span>
          </span>
          <input class="form-control"
            name="newPasswordConfirm"
            @keyup.enter="updatePassword()"
            @input="passwordChanged = true"
            v-model="newPasswordConfirm"
            type="password"
          />
        </div>
      </div> <!-- /password -->

      <!-- notifiers -->
      <div v-if="visibleTab === 'notifiers' && hasAuth && loggedIn"
        class="col">
        <h3>
          Notifiers
        </h3>
        <hr>
        <div class="row">
          <div class="col-12 col-xl-6"
            v-for="notifier of settings.notifiers"
            :key="notifier.name">
            <div class="card bg-light mb-3">
              <div class="card-body">
                <!-- notifier title -->
                <h4 class="mb-3">
                  {{ notifier.name }}
                  <span v-if="!notifier.on"
                    @click="toggleNotifier(notifier)"
                    class="fa fa-toggle-off fa-lg pull-right cursor-pointer"
                    title="Turn this notifier on"
                    v-b-tooltip.hover.bottom-right>
                  </span>
                  <span v-if="notifier.on"
                    @click="toggleNotifier(notifier)"
                    class="fa fa-toggle-on fa-lg pull-right cursor-pointer text-success"
                    title="Turn this notifier off"
                    v-b-tooltip.hover.bottom-right>
                  </span>
                </h4> <!-- /notifier title -->
                <!-- notifier fields -->
                <div class="input-group mb-2"
                  v-for="field of notifier.fields"
                  :key="field.name">
                  <span class="input-group-prepend cursor-help"
                    :title="field.description"
                    v-b-tooltip.hover.bottom-left>
                    <span class="input-group-text">
                      {{ field.name }}
                      <sup v-if="field.required">*</sup>
                    </span>
                  </span>
                  <input class="form-control"
                    v-model="field.value"
                    @input="debounceInput"
                    :type="getFieldInputType(field)"
                  />
                  <span v-if="field.secret"
                    class="input-group-append cursor-pointer"
                    @click="toggleVisibleSecretField(field)">
                    <span class="input-group-text">
                      <span class="fa"
                        :class="{'fa-eye':field.secret && !field.showValue, 'fa-eye-slash':field.secret && field.showValue}">
                      </span>
                    </span>
                  </span>
                </div> <!-- /notifier fields -->
                <!-- notifier actions -->
                <div class="row mt-3">
                  <div class="col-12">
                    <a class="btn btn-sm btn-outline-warning cursor-pointer"
                      @click="clearNotifierFields(notifier)">
                      <span class="fa fa-ban">
                      </span>&nbsp;
                      Clear fields
                    </a>
                    <a class="btn btn-sm btn-outline-success cursor-pointer pull-right"
                      @click="testNotifier(notifier)">
                      <span class="fa fa-bell">
                      </span>&nbsp;
                      Test Notifier
                    </a>
                  </div>
                </div> <!-- /notifier actions -->
                <hr>
                <!-- notifier alerts -->
                <h5>Notify on</h5>
                <div class="row">
                  <div class="col-12">
                    <div v-for="alert of notifier.alerts"
                      :key="alert.name"
                      class="form-check form-check-inline"
                      :title="`Notify if ${alert.description}`"
                      v-b-tooltip.hover.top>
                      <label class="form-check-label">
                        <input class="form-check-input"
                          type="checkbox"
                          @input="updateAlert(alert)"
                          :id="alert.id"
                          :name="alert.id"
                          v-model="alert.on"
                        />
                        {{ alert.name }}
                      </label>
                    </div>
                  </div>
                </div> <!-- /notifier alerts -->
              </div>
            </div>
          </div>
        </div>
      </div> <!-- /notifiers -->

    </div> <!-- /page content -->

  </div>

</template>

<script>
import AuthService from '../auth';
import SettingsService from './settings.service';

let initialized;
let inputDebounce;

export default {
  name: 'Settings',
  data: function () {
    return {
      // page error
      error: '',
      // page success message
      success: '',
      // default tab
      visibleTab: 'password',
      // page data
      settings: {},
      // password settings
      currentPassword: '',
      newPassword: '',
      newPasswordConfirm: '',
      passwordChanged: false
    };
  },
  computed: {
    // auth vars
    hasAuth: function () {
      return this.$store.state.hasAuth;
    },
    loggedIn: function () {
      return this.$store.state.loggedIn;
    }
  },
  watch: {
    loggedIn: function (newVal) {
      if (newVal && initialized) { this.loadData(); }
    }
  },
  mounted: function () {
    // does the url specify a tab in hash
    let tab = window.location.hash;
    if (tab) { // if there is a tab specified and it's a valid tab
      tab = tab.replace(/^#/, '');
      if (tab === 'general' || tab === 'notifiers' || tab === 'password') {
        this.visibleTab = tab;
      }
    }

    this.loadData();
  },
  methods: {
    /* page functions ------------------------------------------------------ */
    openView: function (tabName) {
      this.visibleTab = tabName;
      this.$router.push({
        hash: tabName
      });
    },
    updateAlert: function (alert) {
      alert.on = !alert.on;
      this.saveSettings();
    },
    saveSettings: function () {
      SettingsService.saveSettings(this.settings)
        .then((data) => {
          this.error = '';
        })
        .catch((error) => {
          this.error = error.text || 'Error saving settings.';
        });
    },
    toggleNotifier: function (notifier) {
      this.$set(notifier, 'on', !notifier.on);
      this.saveSettings();
    },
    testNotifier: function (notifier) {
      SettingsService.testNotifier(notifier.name)
        .then((data) => {
          this.error = '';
          this.success = data.text || 'Successfully issued alert.';
          this.closeSuccess();
        })
        .catch((error) => {
          this.error = error.text || 'Error issuing alert.';
        });
    },
    clearNotifierFields: function (notifier) {
      for (const field of notifier.fields) {
        field.value = '';
      }
      this.saveSettings();
    },
    getFieldInputType: function (field) {
      return (field.secret && !field.showValue) ? 'password' : 'text';
    },
    cancelChangePassword: function () {
      this.currentPassword = '';
      this.newPassword = '';
      this.newPasswordConfirm = '';
      this.passwordChanged = false;
    },
    updatePassword: function () {
      if (!this.currentPassword && this.hasAuth) {
        this.error = 'You must provide your current password.';
      }

      if (!this.newPassword) {
        this.error = 'You must provide a new password.';
        return;
      }

      if (!this.newPasswordConfirm) {
        this.error = 'You must confirm your new password.';
        return;
      }

      if (this.newPassword !== this.newPasswordConfirm) {
        this.error = 'Passwords must match.';
        this.newPassword = '';
        this.newPasswordConfirm = '';
        return;
      }

      let success = 'Password successfully ';
      success += this.hasAuth ? 'updated' : 'created';

      AuthService.updatePassword(this.currentPassword, this.newPassword)
        .then((response) => {
          this.error = '';
          this.success = success;
          this.closeSuccess();
          this.cancelChangePassword();
        })
        .catch((error) => {
          this.error = error.text || 'Error saving password.';
          this.cancelChangePassword();
        });
    },
    debounceInput: function () {
      if (inputDebounce) { clearTimeout(inputDebounce); }
      inputDebounce = setTimeout(() => {
        this.saveSettings();
      }, 500);
    },
    toggleVisibleSecretField: function (field) {
      this.$set(field, 'showValue', !field.showValue);
    },
    /* helper functions ---------------------------------------------------- */
    loadData: function () {
      SettingsService.getSettings()
        .then((data) => {
          initialized = true;
          this.error = '';
          this.settings = data;
        })
        .catch((error) => {
          initialized = true;
          if (this.hasAuth) {
            this.error = error.text || 'Error fetching settings.';
          } else {
            this.error = 'No password set for your Parliament. Please set a password so you can do more stuff!';
            this.openView('password'); // redirect the user to possibly create a password
          }
        });
    },
    closeSuccess: function (time) {
      setTimeout(() => {
        this.success = '';
      }, time || 10000);
    }
  }
};
</script>
