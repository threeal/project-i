<template>
  <v-container>
    <v-row justify="center">
      <v-col cols="12" sm="8" md="6">
        <v-row>
          <v-col cols="6">
            <v-text-field
              v-model="basketNumber"
              label="No Basket"
              type="number"
              :disabled="fetching || submitting"
              :loading="fetching"
              :readonly="!edit"
              :filled="!edit"
              :clearable="edit"
              hide-details
              dense
              outlined
            />
          </v-col>
          <v-col cols="6">
            <v-text-field
              v-model="basketId"
              label="ID Basket"
              type="number"
              :disabled="fetching || submitting"
              :loading="fetching"
              :readonly="!edit"
              :filled="!edit"
              :clearable="edit"
              hide-details
              dense
              outlined
            />
          </v-col>
          <v-col cols="6">
            <v-text-field
              v-model="startTime"
              label="Waktu Mulai"
              type="time"
              :disabled="fetching || submitting"
              :loading="fetching"
              :readonly="!edit"
              :filled="!edit"
              hide-details
              dense
              outlined
            />
          </v-col>
          <v-col cols="6">
            <v-text-field
              v-model="endTime"
              label="Waktu Selesai"
              type="time"
              :disabled="fetching || submitting"
              :loading="fetching"
              :readonly="!edit"
              :filled="!edit"
              hide-details
              dense
              outlined
            />
          </v-col>
          <v-col
            v-if="edit || (trayQuantity && trayQuantity > 0)"
            :cols="edit || (canQuantity && canQuantity > 0) ? 6 : 12"
          >
            <v-text-field
              v-model="trayQuantity"
              label="Jumlah Tray"
              type="number"
              :disabled="fetching || submitting"
              :loading="fetching"
              :readonly="!edit"
              :filled="!edit"
              :clearable="edit"
              hide-details
              dense
              outlined
            />
          </v-col>
          <v-col
            v-if="edit || (canQuantity && canQuantity > 0)"
            :cols="edit || (trayQuantity && trayQuantity > 0) ? 6 : 12"
          >
            <v-text-field
              v-model="canQuantity"
              label="Jumlah Kaleng"
              type="number"
              :disabled="fetching || submitting"
              :loading="fetching"
              :readonly="!edit"
              :filled="!edit"
              :clearable="edit"
              hide-details
              dense
              outlined
            />
          </v-col>
          <v-col
            v-if="edit || (rejectQuantity && rejectQuantity > 0)"
            cols="12"
          >
            <v-text-field
              v-model="rejectQuantity"
              label="Jumlah Rijek"
              :disabled="fetching || submitting"
              :loading="fetching"
              :readonly="!edit"
              :filled="!edit"
              :clearable="edit"
              hide-details
              dense
              outlined
            />
          </v-col>
          <v-col v-if="rejectQuantity && rejectQuantity > 0" cols="12">
            <v-select
              v-if="edit"
              v-model="rejectKind"
              :items="rejectKindList"
              label="Jenis Rijek"
              item-text="name"
              item-value="id"
              :disabled="submitting"
              hide-details
              dense
              outlined
            />
            <v-text-field
              v-else
              label="Jenis Rijek"
              :value="rejectKindValue"
              :loading="fetching"
              :disabled="fetching"
              readonly
              filled
              hide-details
              dense
              outlined
            />
          </v-col>
          <v-col cols="12">
            <v-card>
              <v-toolbar color="primary" dark flat dense>
                <v-toolbar-title>Kondisi</v-toolbar-title>
              </v-toolbar>
              <v-card-text>
                <div v-if="fetching" class="d-flex justify-center">
                  <v-progress-circular color="primary" indeterminate />
                </div>
                <v-row v-else>
                  <v-col>
                    <v-checkbox
                      v-model="seamingCondition"
                      label="Seaming"
                      :off-icon="edit ? 'mdi-close-box' : 'mdi-close-thick'"
                      :on-icon="edit ? 'mdi-checkbox-marked' : 'mdi-check-bold'"
                      :readonly="!edit"
                      :disabled="submitting"
                      hide-details
                    />
                  </v-col>
                  <v-col>
                    <v-checkbox
                      v-model="canMarkCondition"
                      label="Can Mark"
                      :off-icon="edit ? 'mdi-close-box' : 'mdi-close-thick'"
                      :on-icon="edit ? 'mdi-checkbox-marked' : 'mdi-check-bold'"
                      :readonly="!edit"
                      :disabled="submitting"
                      hide-details
                    />
                  </v-col>
                  <v-col>
                    <v-checkbox
                      v-model="indicatorCondition"
                      label="Indicator"
                      :off-icon="edit ? 'mdi-close-box' : 'mdi-close-thick'"
                      :on-icon="edit ? 'mdi-checkbox-marked' : 'mdi-check-bold'"
                      :readonly="!edit"
                      :disabled="submitting"
                      hide-details
                    />
                  </v-col>
                </v-row>
              </v-card-text>
            </v-card>
          </v-col>
          <v-col cols="12">
            <v-btn
              v-if="!edit"
              @click="onEdit()"
              :disabled="fetching"
              color="primary"
              block
            >
              <v-icon left>mdi-pencil</v-icon> Ubah Detail
            </v-btn>
            <v-btn
              v-else
              @click="onSave()"
              :disabled="submitDisabled"
              :loading="submitting"
              color="success"
              block
            >
              <v-icon left>mdi-content-save</v-icon> Simpan Perubahan
            </v-btn>
          </v-col>
          <v-col cols="12">
            <v-btn @click="onDelete()" :disabled="fetching" color="error" block>
              <v-icon left>mdi-delete</v-icon> Hapus Data Basket
            </v-btn>
          </v-col>
        </v-row>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import BasketService from "../services/BasketService";
import AuthService from "../services/AuthService";

export default {
  name: "basket-detail",
  props: {
    app: { type: Object, required: true }
  },
  data() {
    return {
      fetching: true,
      submitting: false,
      edit: false,
      basketNumber: null,
      basketId: null,
      startTime: null,
      endTime: null,
      trayQuantity: null,
      canQuantity: null,
      rejectQuantity: null,
      rejectKind: null,
      rejectKindList: [
        { id: "PJ", name: "Penyok Jatuh (PJ)" },
        { id: "TJ", name: "Terjepit (TJ)" },
        { id: "SB", name: "Score Break (SB)" },
        { id: "PB", name: "Penyok dari Basket (PB)" },
        { id: "FD", name: "Flange Down (FD)" }
      ],
      seamingCondition: null,
      canMarkCondition: null,
      indicatorCondition: null
    };
  },
  computed: {
    rejectKindValue() {
      let rejectKindObj = this.rejectKindList.find(
        o => o.id == this.rejectKind
      );
      if (rejectKindObj) {
        return rejectKindObj.name;
      } else {
        return "-";
      }
    },
    submitDisabled() {
      return (
        this.submitting ||
        !this.basketNumber ||
        !this.basketId ||
        !this.startTime ||
        !this.endTime ||
        (!this.trayQuantity && !this.canQuantity) ||
        (this.rejectQuantity > 0 && !this.rejectKind)
      );
    }
  },
  methods: {
    onEdit() {
      this.edit = true;
    },
    onSave() {
      this.submitting = true;

      let data = {
        basketNumber: this.basketNumber,
        basketId: this.basketId,
        startTime: this.startTime,
        endTime: this.endTime,
        trayQuantity: this.trayQuantity,
        canQuantity: this.canQuantity,
        rejectQuantity: this.rejectQuantity,
        rejectKind: this.rejectKind,
        seamingCondition: this.seamingCondition || false,
        canMarkCondition: this.canMarkCondition || false,
        indicatorCondition: this.indicatorCondition || false
      };

      BasketService.update(
        this.$route.params.documentId,
        this.$route.params.basketId,
        data
      )
        .then(() => {
          this.app.log("Detail data basket berhasil diperbaharui");
          this.edit = false;
          this.submitting = false;
        })
        .catch(err => {
          this.submitting = false;

          if (err.response) {
            if (err.response.status === 401) {
              this.app.log("Sesi habis, harap masuk kembali");

              AuthService.signOut();
              this.app.routeReplace("/login");
            } else {
              this.app.log(
                "Detail data basket gagal diperbaharui," +
                  ` kesalahan server (${err.response.status})`
              );
            }
          } else {
            this.app.log(
              "Detail data basket gagal diperbaharui, tidak ada jaringan"
            );
          }
        });
    },
    onDelete() {
      this.app.confirm({
        description: "Apakah anda yakin ingin menghapus data basket ini?",
        promiseCallback: () => {
          return BasketService.remove(
            this.$route.params.documentId,
            this.$route.params.basketId
          );
        },
        thenCallback: () => {
          this.app.log("Data basket berhasil dihapus");
          this.$router.go(-1);
        },
        catchCallback: err => {
          if (err.response) {
            if (err.response.status === 401) {
              this.app.log("Sesi habis, harap masuk kembali");

              AuthService.signOut();
              this.app.routeReplace("/login");
            } else {
              this.app.log(
                "Data basket gagal dihapus," +
                  ` kesalahan server (${err.response.status})`
              );
            }
          } else {
            this.app.log("Data basket gagal dihapus, tidak ada jaringan");
          }
        }
      });
    }
  },
  created() {
    this.app.setAppBar(true, "Detail Data Basket");
  },
  mounted() {
    BasketService.findOne(
      this.$route.params.documentId,
      this.$route.params.basketId
    )
      .then(res => {
        this.basketNumber = res.data.basketNumber;
        this.basketId = res.data.basketId;
        this.startTime = res.data.startTime;
        this.endTime = res.data.endTime;
        this.trayQuantity = res.data.trayQuantity;
        this.canQuantity = res.data.canQuantity;
        this.rejectQuantity = res.data.rejectQuantity;
        this.rejectKind = res.data.rejectKind;
        this.seamingCondition = res.data.seamingCondition || false;
        this.canMarkCondition = res.data.canMarkCondition || false;
        this.indicatorCondition = res.data.indicatorCondition || false;

        this.fetching = false;
      })
      .catch(err => {
        if (err.response) {
          if (err.response.status === 401) {
            this.app.log("Sesi habis, harap masuk kembali");

            AuthService.signOut();
            this.app.routeReplace("/login");
          } else {
            this.app.log(
              "Gagal mengambil detail data basket," +
                ` kesalahan server (${err.response.status})`
            );
          }
        } else {
          this.app.log(
            "Gagal mengambil detail data basket, tidak ada jaringan"
          );
        }
      });
  }
};
</script>
