<template>
  <v-select
    :id="name"
    icon="perm_identity"
    :name="name"
    :placeholder="$t('interfaces-user-roles-choose_role')"
    :options="selectOptions"
    :value="typeof value === 'object' ? value.id : value"
    @input="emitValue"
  ></v-select>
</template>

<script>
import mixin from "@directus/extension-toolkit/mixins/interface";

export default {
  name: "InterfaceUserRole",
  mixins: [mixin],
  data() {
    return {
      loading: false,
      roles: [],
      error: null
    };
  },
  computed: {
    selectOptions() {
      const options = {};

      this.roles
        .filter(role => {
          if (this.options.showPublic) return true;

          return role.id !== 2;
        }) // 2 = public role
        .forEach(role => {
          options[role.id] = role.name;
        });

      return options;
    }
  },
  created() {
    this.fetchRoles();
  },
  methods: {
    fetchRoles() {
      this.loading = true;

      this.$api
        .getRoles()
        .then(res => res.data)
        .then(roles => {
          this.roles = roles;
          this.loading = false;
          this.error = null;
        })
        .catch(error => {
          this.loading = false;
          this.error = error;
        });
    },
    emitValue(value) {
      this.$emit("input", value);
    }
  }
};
</script>

<style lang="scss" scoped></style>
