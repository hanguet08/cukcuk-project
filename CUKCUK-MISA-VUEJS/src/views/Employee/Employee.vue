<template>
  <div class="content">
    <!-- header -->
    <div class="content-header">
      <div class="header-title">Danh sách nhân viên</div>
      <button
        id="btnAddEmployee"
        class="m-btn m-btn-bg add-icon"
        @click="open"
      >
        Thêm nhân viên
      </button>
    </div>

    <!-- filter -->
    <div class="content-filter">
      <div class="filter-left">
        <div class="search-by-id">
          <input
            type="text"
            class="search-box input-search"
            placeholder="Tìm kiếm theo Mã, Tên hoặc Số điện thoại"
          />
        </div>
        <div class="combobox" title="Lọc theo phòng ban">
          <combobox-component :items="departments" />
        </div>
        <div class="combobox" title="Lọc theo vị trí">
          <combobox-component :items="positions" />
        </div>
      </div>
      <div class="filter-right">
        <button
          @click="onClickEdit"
          id="btnEdit"
          class="m-btn-bg btn-edit disabled tooltip"
          v-bind:class="{ able: this.employeeSelected.EmployeeId != undefined }"
        >
          <i class="fas fa-edit"></i>
          <span class="tooltiptext">Sửa</span>
        </button>

        <button
          @click="onClickDelete"
          id="btnDelete"
          class="m-btn-bg btn-delete disabled tooltip"
          v-bind:class="{ able: this.employeeSelected.EmployeeId != undefined }"
        >
          <i class="fas fa-trash-alt"></i>
          <span class="tooltiptext">Xóa</span>
        </button>

        <button
          id="btnRefresh"
          class="m-btn-bg refresh-icon btn-refresh tooltip"
        >
          <span class="tooltiptext refresh-text">Làm mới</span>
        </button>
      </div>
    </div>

    <!-- start table -->
    <div class="content-table">
      <table id="tableEmployee" class="table">
        <thead>
          <tr>
            <!-- <th><input type="checkbox" value="" /></th> -->
            <th style="padding-right: 0">
              <div ref="checkbox" class="m-checkbox">
                <i class="fas fa-check"></i>
              </div>
            </th>
            <th fieldName="EmployeeCode">Mã nhân viên</th>
            <th fieldName="EmployeeName" colspan="2">Họ và tên</th>
            <th style="width: 50px" fieldName="Gender">Giới tính</th>
            <th class="text-align-center" fieldName="DateOfBirth">Ngày sinh</th>
            <th fieldName="PhoneNumber">Số điện thoại</th>
            <th fieldName="Email" colspan="2">Email</th>
            <!-- <th fieldName="PositionName">Chức vụ</th> -->
            <th fieldName="DepartmentName" colspan="2">Phòng ban</th>
            <th class="text-align-right" fieldName="Salary">
              Mức lương cơ bản
            </th>
            <!-- <th class="text-align-center">Chức năng</th> -->
            <!-- <th>Tình trạng công việc</th> -->
          </tr>
        </thead>
        <tbody v-show="!isShowLoading">
          <!-- <router-view /> -->
          <div>
            <tr
              @click="onClickRow($event, employee)"
              @dblclick="onDblclickRow()"
              v-bind:class="{
                selectedRow:
                  this.employeeSelected.EmployeeId == employee.EmployeeId,
              }"
              v-for="employee in employees"
              :key="employee.EmployeeId"
            >
              <!-- <td><input type="checkbox" value="" /></td> -->
              <td>
                <div ref="checkbox" class="m-checkbox" 
                  v-bind:class="{
                    checked:
                      this.employeeSelected.EmployeeId == employee.EmployeeId,
                  }">
                  <i class="fas fa-check"></i>
                </div>
              </td>
              <td>{{ employee.EmployeeCode }}</td>
              <td colspan="2">{{ employee.EmployeeName }}</td>
              <td style="width: 50px">
                {{ getValueEnum(employee.Gender, "Gender") }}
              </td>
              <td class="align-center">
                {{ formatDate(employee.DateOfBirth) }}
              </td>
              <td>{{ employee.PhoneNumber }}</td>
              <td colspan="2">{{ employee.Email }}</td>
              <!-- <td></td> -->
              <td colspan="2">{{ employee.DepartmentName }}</td>
              <td class="align-right">{{ formatMoney(1000000) }}</td>
              <!-- <td class="text-align-center action feature">
                <i
                @click="rowOnClick(emp)"
                style="font-size: 17px; color: #138496; margin-right: 10px"
                class="far fa-edit edit-form-employee"
              ></i>
              <i
                v-on:click="deleteEmployee(emp)"
                style="font-size: 17px; color: red"
                class="text-align-center far fa-trash-alt btnDelete"
              ></i>
              </td> -->
            </tr>
          </div>
        </tbody>
        <TheLoading v-show="isShowLoading" />
      </table>
    </div>
    <!-- end table -->
    <div class="summary">

    </div>
    <!-- paging bar -->
    <div class="paging-bar">
      <div class="paging-left">Hiển thị 1-10/1000 nhân viên</div>
      <div class="pagination">
        <button class="btn-page firstpage-icon"></button>
        <button class="btn-page prevpage-icon"></button>
        <div class="pagination-number">
          <button class="btn-pagenumber cur-active">1</button>
          <button class="btn-pagenumber">2</button>
          <button class="btn-pagenumber">3</button>
          <button class="btn-pagenumber">4</button>
        </div>
        <button class="btn-page nextpage-icon"></button>
        <button class="btn-page lastpage-icon"></button>
      </div>
      <div class="paging-right">10 nhân viên/trang</div>
    </div>

    <!-- Form thêm mới hoặc cập nhật thông tin nhân viên -->
    <employee-detail
      :isShow="form.isShowForm"
      :employee="employeeSelected"
      :formMode="form.formMode"
      @closeForm="closeForm"
      @handleSave="handleSave"
      @validateForm="validateForm"
    />

    <!-- Dialog xác nhận xóa hoặc cảnh báo -->
    <the-dialog
      v-bind:isShowDialog="dialog.isShowDialog"
      v-bind:title="dialog.title"
      v-bind:message="dialog.message"
      v-bind:isShowBtnCancel="dialog.isShowBtnCancel"
      v-bind:employeeCode="dialog.employeeCode || ``"
      @handleCloseDialog="handleCloseDialog"
      @handleConfirmDialog="handleConfirmDialog"
    />

    <!-- Toast message thông báo thành công -->
    <the-toast-message
      @handleCloseToast="handleCloseToast"
      v-bind:isShowToast="toast.isShowToast"
      v-bind:message="toast.message"
    />
  </div>
</template>

<script>
/* eslint-disable */
import EmployeeDetail from "./EmployeeDetail.vue";
import TheLoading from "../../components/base/TheLoading.vue";
import TheDialog from "../../components/base/TheDialog.vue";
import TheToastMessage from "../../components/base/ToastMessage.vue";
import CommonFn from "../../js/Common/Common.js";
import Combobox from "../../js/Components/Combobox.js";
import DialogJS from "../../js/Components/Dialog.js";
import Resource from "../../js/Common/Resource";
import Constant from "../../js/Common/Constant";
import Enumeration from "../../js/Common/Enumeration";
import ComboboxComponent from "../../components/base/Combobox.vue";
import $ from 'jquery'

export default {
  name: "TheEmployee",
  components: {
    EmployeeDetail,
    TheDialog,
    TheToastMessage,
    TheLoading,
    ComboboxComponent,
  },
  props: [],
  data() {
    return {
      /*
        Phục vụ chung
        CreateBy : Nguyễn Văn Khang 30/3/2022
      */
      employees: { type: Object, default: {} },
      employeeSelected: {},
      isShowLoading: { type: Boolean, default: false },

      // Phục vụ cho Form nhân viên
      form: {
        isShowForm: false,
        formMode: { type: Number, default: 0 },
      },

      // Phục vụ cho Dialog
      dialog: {
        isShowDialog: false,
        title: "",
        message: "",
        isShowBtnCancel: false,
      },

      // Phục vụ cho Toast message
      toast: {
        isShowToast: false,
        message: { type: String, default: "" },
      },

      // Lấy dữ liệu từ server và build combobox
      departments: Combobox.getDepartment("Employee"),
      positions: Combobox.getPosition("Employee"),
    };
  },

  methods: {
    // Các hàm format 
    formatDate: CommonFn.formatDate,
    convertDate: CommonFn.convertDate,
    getValueEnum: CommonFn.getValueEnum,
    formatMoney: CommonFn.formatMoney,

    // mở form detail
    open() {
      let me = this;
      me.form = {
        isShowForm: true,
        formMode: Enumeration.FormMode.Add,
      };
      me.employee = {};
    },

    // Đóng form detail
    closeForm() {
      let me = this;
      me.form = {
        isShowForm: false,
        formMode: 0,
      };
    },

    // Sự kiện select row
    onClickRow(event, employeeSelected) {
      let me = this;
      me.employeeSelected = employeeSelected;
    },

    // Sự kiện double select row
    onDblclickRow() {
      let me = this;
      me.form = {
        isShowForm: true,
        formMode: Enumeration.FormMode.Edit,
      };
    },

    // Sự kiện click button Delete
    onClickDelete() {
      let me = this;
      if (me.employeeSelected.EmployeeCode != undefined) {
        me.dialog = DialogJS.Delete(me.employeeSelected.EmployeeCode);
      } else {
        console.log("Chưa chọn employee nào");
      }
    },

    // Sự kiện click vào button Edit
    onClickEdit() {
      let me = this;
      if (me.employeeSelected.EmployeeCode != undefined) {
        me.onDblclickRow();
      } else {
        console.log("Chưa chọn employee nào");
      }
    },

    // Lưu dữ liệu
    handleSave(employeeForm) {
      var me = this,
        method = null,
        url = Constant.urlEmployee,
        data = employeeForm,
        message = "";

      switch (me.form.formMode) {
        case Enumeration.FormMode.Add:
          method = Resource.Method.Post;
          message = Resource.Toast.Add;
          break;
        case Enumeration.FormMode.Edit:
          method = Resource.Method.Put;
          message = Resource.Toast.Edit;
          url = `${url}/${employeeForm.EmployeeId}`;
          break;
      }
      // call axios
      CommonFn.Axios(method, url, data, function (response) {
        if (response != null) {
          // Load lại dữ liệu table
          me.loadingEmployees();

          // Đóng Form
          me.closeForm();

          // Hiển thị Toast message
          me.showToast(message);
        }
      }, function (error){
        // hiển thị dialog thông báo lỗi đc gửi từ server
        me.dialog = DialogJS.Warning(error.response.data.devMsg);
      });
    },

    // Hiển thị thông báo lỗi
    validateForm(message) {
      let me = this;
      me.dialog = DialogJS.Warning(message);
    },

    // Hàm xử lý sự kiện click CONFIRM DELETE button
    handleConfirmDialog() {
      let me = this,
        url = "";
      if (me.dialog.isShowBtnCancel == false) {
        me.handleCloseDialog();
      } else {
        url = `${Constant.urlEmployee}/${me.employeeSelected.EmployeeId}`;
        // call axios delete
        CommonFn.Axios(Resource.Method.Delete, url, {}, function (response) {
          console.log(response);
          me.loadingEmployees();
          me.showToast(Resource.Toast.Delete);
        });
      }
      // Đóng dialog delete và reset
      me.handleCloseDialog();
    },

    // Hàm xử lý sự kiện click CANCEL DELETE button
    handleCloseDialog() {
      let me = this;
      me.dialog = {
        isShowDialog: false,
        //employeeCode: null,
        isShowBtnCancel: false,
        title: "",
        message: "",
      };
    },

    // Đóng toast message
    handleCloseToast() {
      let me = this;
      me.toast = {
        isShowToast: false,
      };
    },

    // Hiển thị toast message
    showToast(message) {
      let me = this;
      (me.toast = {
        isShowToast: true,
        message,
      }),
        setTimeout(() => {
          me.toast = {
            isShowToast: false,
          };
        }, 3000);
    },

    // Hàm load lại dữ liệu trên table
    loadingEmployees() {
      let me = this,
        method = Resource.Method.Get,
        url = Constant.urlEmployee,
        data = {};

      // hiện image loading
      me.isShowLoading = true;

      // call axios lấy dữ liệu
      CommonFn.Axios(method, url, data, function (response) {
        // load dữ liệu vào bảng
        me.employees = response.data;
      });

      // Reset lại item được chọn
      me.employeeSelected = {};
      setTimeout(() => {
        me.isShowLoading = false;
      }, 300);
    },

    // Sự kiện click vào checkbox
    checkboxOnClick(trSelected){
      let tr = $('tbody tr .m-checkbox');
      $.each(tr, (item) => {
        if($(item).hasClass('checked')){
          $(item).removeClass('checked');
        }
      })
      $(trSelected).addClass('checked');
    },
  },

  /* 
    2. created
  */
  created() {
    console.log("Created");
    let me = this;
    // Tải dữ liệu lên bảng
    me.loadingEmployees();
  },
};
</script>

<style>
.align-center {
  text-align: center;
}
.align-right {
  text-align: right;
}
#btnDelete.disabled,
#btnEdit.disabled,
.disabled:hover {
  background-color: #eeeeee;
  cursor: context-menu;
}

#btnDelete.able,
#btnEdit.able {
  background-color: white;
  cursor: pointer;
}

#btnDelete.able:hover,
#btnEdit.able:hover {
  background-color: #eeeeee;
}
#btnDelete,
#btnEdit {
  position: relative;
}

.refresh-text {
  margin-left: -100px;
}
</style>
