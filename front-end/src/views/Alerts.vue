<template>
  <div>
    <b-table
      hover
      bordered
      show-empty
      responsive
      :busy="isBusy"
      :items="alerts"
      :fields="fields"
    >
      <template #cell(icon)="{ item }">
        <img width="30" height="30" :src="item.item.icon" />
      </template>
      <template #cell(name)="{ item }">
        <span>{{ item.item.name }} </span>
      </template>
      <template #cell(matches)="{ item }">
        <span>{{ item.matches.length }} </span>
      </template>
      <template #cell(actions)="props">
        <div>
          <b-button
            v-if="props.item.matches.length"
            pill
            size="sm"
            variant="primary"
            @click="props.toggleDetails"
          >
            {{ props.toggleDetails ? "Hide Matches" : "Show Matches" }}
          </b-button>
          <b-button pill size="sm" @click="onClickEditAlert(props.item)">
            Edit
          </b-button>
          <b-button
            pill
            size="sm"
            variant="danger"
            @click="onClickDelete(props.item)"
          >
            <b-spinner small v-if="props.item.isDeleting" />
            <span v-else> Delete </span>
          </b-button>
        </div>
      </template>
      <template #row-details="{ item }">
        <b-card>
          <b-row class="font-weight-bold">
            <b-col> Shop name </b-col>
            <b-col> Owner name </b-col>
            <b-col>Amount</b-col>
            <b-col> Map </b-col>
            <b-col> Location </b-col>
          </b-row>
          <b-row :key="match.id" v-for="match in item.matches">
            <b-col>
              {{ match.shop_name }}
            </b-col>

            <b-col>
              {{ match.owner_name }}
            </b-col>
            <b-col>
              {{ match.amount }}
            </b-col>
            <b-col>
              {{ match.map }}
            </b-col>
            <b-col>
              {{ `X: ${match.location_x} | Y: ${match.location_y}` }}
            </b-col>
          </b-row>
        </b-card>
      </template>
    </b-table>
    <AlertForm
      :show="form.show"
      :data="form.data"
      :isSubmitting="form.isSubmitting"
      @submit="onSubmitAlert"
      @cancel="onCancelAlertEdit"
    />
  </div>
</template>
<script>
import http from "../services/http";
import AlertForm from "./components/AlertForm";
export default {
  components: {
    AlertForm,
  },
  data() {
    return {
      fields: [
        {
          key: "item_id",
          label: "Item ID",
          thClass: "text-center",
          tdClass: "text-center",
        },
        {
          key: "icon",
          label: "Icon",
          thClass: "text-center",
          tdClass: "text-center",
        },
        {
          key: "name",
          label: "Name",
          thClass: "text-center",
          tdClass: "text-center",
        },
        {
          key: "max_price",
          label: "Max Price",
          thClass: "text-center",
          tdClass: "text-center",
        },
        {
          key: "matches",
          label: "Found",
          thClass: "text-center",
          tdClass: "text-center",
        },
        {
          key: "actions",
          label: "Actions",
          thClass: "text-center",
          tdClass: "text-center",
        },
      ],
      isBusy: false,
      alerts: [],
      form: {
        data: {},
        show: false,
        isSubmitting: false,
      },
    };
  },
  mounted() {
    this.getAlerts();
  },
  methods: {
    onSubmitAlert(data) {
      this.form.isSubmitting = true;
      const { max_price, alert } = data;

      http
        .updateAlert(alert.id, { max_price })
        .then(({ data }) => {
          if (data.updated) {
            alert.max_price = max_price;
          }
        })
        .finally(() => {
          this.form.isSubmitting = false;
          this.form.show = false;
        });
    },
    onCancelAlertEdit() {
      this.form.show = false;
    },
    onClickEditAlert(alert) {
      this.setFormData(alert);
      this.form.show = true;
    },
    setFormData(alert) {
      this.form.data = {
        alert,
        ...alert.item,
        max_price: alert.max_price,
      };
    },
    onClickDelete(alert) {
      alert.isDeleting = true;

      http
        .deleteAlert(alert.id)
        .then(() => {
          this.showSuccessToast();
          this.remoteAlert(alert);
        })
        .catch(() => {
          this.showErrorToast();
        })
        .finally(() => {
          alert.isDeleting = false;
        });
    },
    showSuccessToast() {
      this.$bvToast.toast(`The alert was deleted successfully.`, {
        title: "Success!",
        variant: "success",
      });
    },
    showErrorToast() {
      this.$bvToast.toast(`Something goes wrong. Try again.`, {
        title: "Error!",
        variant: "danger",
      });
    },
    remoteAlert(item) {
      this.alerts = this.alerts.filter((i) => i != item);
    },
    getAlerts() {
      this.isBusy = true;
      http
        .getAlerts()
        .then((resp) => {
          this.alerts = resp.data.map((i) => {
            return {
              ...i,
              isDeleting: false,
              _showDetails: i.matches.length > 0,
            };
          });
        })
        .finally(() => {
          this.isBusy = false;
        });
    },
  },
};
</script>
<style scoped>
.col {
  text-align: center;
}
</style>
