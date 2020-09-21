<template>
  <q-table
    :data="kiosks"
    :columns="[{
                 name: 'authorised',
                 label: 'Authorised',
                 field: 'authorised',
                 sortable: true,
                 format: (val, row) => this.capitaliseFirst(val) },
               { name: 'name', label: 'Name', field: 'name', sortable: true },
               { name: 'lastSeen',
                 label: 'Last Seen',
                 field: 'lastSeen',
                 sortable: true,
                 format: (val, row) => this.formatDate(val)
               },
    ]"
    row-key="id"
    :filter="filter"
    :pagination.sync="pagination"
    :loading="loading"
    :grid="$q.screen.xs"
    :no-data-label="$t('kiosk.nodata')"
  >
    <template v-slot:top-right>
      <q-input
        outlined
        dense
        debounce="300"
        v-model="filter"
        placeholder="Search"
      >
        <template v-slot:append>
          <q-icon :name="icons.search" />
        </template>
      </q-input>
    </template>

    <template v-slot:header="props">
      <q-tr :props="props">
        <q-th auto-width />
        <q-th
          v-for="col in props.cols"
          :key="col.name"
          :props="props"
        >
          {{ col.label }}
        </q-th>
        <q-th auto-width>
          {{ $t('edit') }}
        </q-th>
        <q-th auto-width>
          {{ $t('delete') }}
        </q-th>
      </q-tr>
    </template>

    <template v-slot:body="props">
      <q-tr :props="props">
        <q-td auto-width>
          <q-btn
            size="sm"
            color="accent"
            round
            @click="props.expand = !props.expand"
            :icon="props.expand ? icons.down : icons.up"
          />
        </q-td>
        <q-td
          v-for="col in props.cols"
          :key="col.name"
          :props="props"
        >
          {{ col.value }}
        </q-td>

        <q-td auto-width>
          <q-btn
            size="sm"
            color="accent"
            round
            @click="editKiosk(props.row.id)"
            :icon="icons.edit"
          />

          <q-dialog v-model="editKioskDialog">
            <q-card>
              <q-card-section class="row items-center q-pb-none dialog-close-button">
                <q-space />
                <q-btn
                  :icon="icons.close"
                  flat
                  round
                  dense
                  v-close-popup
                />
              </q-card-section>

              <q-card-section>
                <kiosk-form :kiosk-id="editKioskId" />
              </q-card-section>
            </q-card>
          </q-dialog>
        </q-td>

        <q-td auto-width>
          <q-btn
            size="sm"
            color="accent"
            round
            @click="deleteKiosk(props.row.id)"
            :icon="icons.delete"
          />
        </q-td>
      </q-tr>
      <q-tr
        v-show="props.expand"
        :props="props"
      >
        <q-td colspan="100%">
          <kiosk-details
            :kiosk="props.row"
          />
        </q-td>
      </q-tr>
    </template>
  </q-table>
</template>

<script>
import { mapActions, mapGetters } from 'vuex';
import KioskForm from 'components/KioskForm';
import KioskDetails from 'components/KioskDetails';
import icons from '../icons';
import formatMixin from '../mixins/formatMixin';

export default {
  name: 'KioskList',
  mixins: [formatMixin],
  components: { KioskForm, KioskDetails },
  data() {
    return {
      loading: false,
      filter: '',
      editKioskId: '',
      editKioskDialog: false,
      pagination: {
        sortBy: 'date',
        descending: true,
        rowsPerPage: this.$q.screen.xs ? 2 : 12,
      },
    };
  },
  methods: {
    ...mapActions('adminTools', ['getKiosks']),
    deleteKiosk(id) {
      this.$q.dialog({
        title: 'Confirm',
        message: this.$t('kiosk.delete'),
        cancel: {
          color: 'primary',
          flat: true,
          label: this.$t('button.cancel'),
        },
        ok: {
          color: 'primary',
          label: this.$t('button.ok'),
        },
        persistent: true,
      }).onOk(() => {
        this.$axios.delete(`/api/kiosks/${id}/`, this.form)
          .then(() => {
            this.getKiosks();
          })
          .catch(() => {
            this.$q.dialog({
              title: this.$t('error.error'),
              message: this.$t('error.requestFailed'),
            });
          });
      });
    },
    editKiosk(id) {
      this.editKioskId = id;
      this.editKioskDialog = true;
    },
  },
  mounted() {
    this.loading = true;
    this.getKiosks()
      .finally(() => {
        this.loading = false;
      });
  },
  computed: {
    ...mapGetters('adminTools', ['kiosks']),
    icons() {
      return icons;
    },
  },
};
</script>

<style lang="stylus" scoped>
  @media (max-width: $breakpoint-xs-max)
    .access-list
      width: 100%;
</style>