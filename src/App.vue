<template>
  <div>
    <div class="w-[80%] m-auto mt-4 h-[100vh]">
      <main class="main">
        <header class="d-flex justify-content-between mb-3">
          <h3>Nhân viên</h3>
          <button class="btn btn-primary" @click="openAddEmployeeForm">
            Thêm mới nhân viên
          </button>
        </header>
        <div class="d-flex align-items-center justify-content-end gap-2 mb-3">
          <input
            style="width: 350px"
            type="text"
            class="form-control"
            v-model="searchEmail"
            placeholder="Tìm kiếm theo email"
          />
          <i
            class="fa-solid fa-arrows-rotate"
            title="Refresh"
            @click="refresh"
          ></i>
        </div>
        <!-- Danh sách nhân viên -->
        <table class="table table-bordered table-hover table-striped">
          <thead>
            <tr>
              <th>STT</th>
              <th>Họ và tên</th>
              <th>Ngày sinh</th>
              <th>Email</th>
              <th>Địa chỉ</th>
              <th>Trạng thái</th>
              <th colspan="3">Chức năng</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="(employee, index) in filteredEmployees"
              :key="employee.email"
            >
              <td>{{ index + 1 }}</td>
              <td>{{ employee.name }}</td>
              <td>{{ employee.dob }}</td>
              <td>{{ employee.email }}</td>
              <td>{{ employee.address }}</td>
              <td>
                <div style="display: flex; align-items: center; gap: 8px">
                  <div
                    :class="
                      employee.status
                        ? 'status status-active'
                        : 'status status-stop'
                    "
                  ></div>
                  <span>{{
                    employee.status ? "Đang hoạt động" : "Ngừng hoạt động"
                  }}</span>
                </div>
              </td>
              <td>
                <button
                  @click="toggleBlock(employee)"
                  class="button button-block"
                >
                  {{ employee.status ? "Chặn" : "Bỏ chặn" }}
                </button>
              </td>
              <td>
                <button
                  @click="editEmployee(employee)"
                  class="button button-edit"
                >
                  Sửa
                </button>
              </td>
              <td>
                <button
                  @click="confirmDelete(employee)"
                  class="button button-delete"
                >
                  Xóa
                </button>
              </td>
            </tr>
          </tbody>
        </table>
        <!-- Pagination and footer -->
        <footer class="d-flex justify-content-end align-items-center gap-3">
          <select class="form-select" v-model="itemsPerPage">
            <option value="10">Hiển thị 10 bản ghi trên trang</option>
            <option value="20">Hiển thị 20 bản ghi trên trang</option>
            <option value="50">Hiển thị 50 bản ghi trên trang</option>
            <option value="100">Hiển thị 100 bản ghi trên trang</option>
          </select>
          <ul class="pagination">
            <li class="page-item">
              <a class="page-link" href="#" @click="prevPage">Previous</a>
            </li>
            <li v-for="page in totalPages" :key="page" class="page-item">
              <a class="page-link" href="#" @click="setPage(page)">{{
                page
              }}</a>
            </li>
            <li class="page-item">
              <a class="page-link" href="#" @click="nextPage">Next</a>
            </li>
          </ul>
        </footer>
      </main>
    </div>

    <!-- Form thêm mới nhân viên -->
    <div v-if="showAddForm" class="overlay">
      <form class="form">
        <div class="d-flex justify-content-between align-items-center">
          <h4>Chỉnh sửa nhân viên</h4>
          <i class="fa-solid fa-xmark" @click="closeForm"></i>
        </div>
        <div>
          <label class="form-label" for="userName">Họ và tên</label>
          <input
            id="userName"
            v-model="formData.name"
            type="text"
            class="form-control"
          />
        </div>
        <div>
          <label class="form-label" for="dateOfBirth">Ngày sinh</label>
          <input
            id="dateOfBirth"
            v-model="formData.dob"
            type="date"
            class="form-control"
          />
        </div>
        <div>
          <label class="form-label" for="email">Email</label>
          <input
            id="email"
            v-model="formData.email"
            type="email"
            class="form-control"
          />
        </div>
        <div>
          <label class="form-label" for="address">Địa chỉ</label>
          <textarea
            class="form-control"
            id="address"
            v-model="formData.address"
            rows="3"
          ></textarea>
        </div>
        <div>
          <button class="w-100 btn btn-primary" @click="addOrUpdateEmployee">
            Lưu
          </button>
        </div>
      </form>
    </div>

    <!-- Modal xác nhận xóa tài khoản -->
    <div v-if="showDeleteConfirm" class="overlay">
      <div class="modal-custom">
        <div class="modal-title">
          <h4>Cảnh báo</h4>
          <i class="fa-solid fa-xmark" @click="closeConfirm"></i>
        </div>
        <div class="modal-body-custom">
          <span>Bạn có chắc chắn muốn xóa tài khoản này?</span>
        </div>
        <div class="modal-footer-custom">
          <button class="btn btn-light" @click="closeConfirm">Hủy</button>
          <button class="btn btn-danger" @click="deleteEmployee">
            Xác nhận
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";

const employees = ref([
  {
    name: "Nguyễn Văn Nam",
    dob: "28/02/1990",
    email: "namnv@gmail.com",
    address: "Ba Đình, Hà Nội",
    status: true,
  },
  {
    name: "Trần Thị Huyền",
    dob: "15/07/1985",
    email: "tthuyen@gmail.com",
    address: "Cầu Giấy, Hà Nội",
    status: false,
  },
  {
    name: "Lê Văn Cao",
    dob: "03/10/2000",
    email: "lvcao@gmail.com",
    address: "Hai Bà Trưng, Hà Nội",
    status: true,
  },
]);

const searchEmail = ref("");
const showAddForm = ref(false);
const showDeleteConfirm = ref(false);
const currentEmployee = ref(null);
const formData = ref({ name: "", dob: "", email: "", address: "" });
const itemsPerPage = ref(10);
const currentPage = ref(1);

// Lọc nhân viên theo email
const filteredEmployees = computed(() =>
  employees.value.filter((e) =>
    e.email.toLowerCase().includes(searchEmail.value.toLowerCase())
  )
);

// Số trang
const totalPages = computed(() =>
  Math.ceil(filteredEmployees.value.length / itemsPerPage.value)
);

// Chuyển trang
const setPage = (page) => {
  currentPage.value = page;
};

const nextPage = () => {
  if (currentPage.value < totalPages.value) currentPage.value++;
};

const prevPage = () => {
  if (currentPage.value > 1) currentPage.value--;
};

const openAddEmployeeForm = () => {
  showAddForm.value = true;
};

const closeForm = () => {
  showAddForm.value = false;
  formData.value = { name: "", dob: "", email: "", address: "" };
};

const addOrUpdateEmployee = () => {
  if (currentEmployee.value) {
    currentEmployee.value.name = formData.value.name;
    currentEmployee.value.dob = formData.value.dob;
    currentEmployee.value.email = formData.value.email;
    currentEmployee.value.address = formData.value.address;
  } else {
    employees.value.push({ ...formData.value, status: true });
  }
  closeForm();
};

const editEmployee = (employee) => {
  formData.value = { ...employee };
  currentEmployee.value = employee;
  openAddEmployeeForm();
};

const confirmDelete = (employee) => {
  currentEmployee.value = employee;
  showDeleteConfirm.value = true;
};

const closeConfirm = () => {
  showDeleteConfirm.value = false;
};

const deleteEmployee = () => {
  employees.value = employees.value.filter(
    (e) => e.email !== currentEmployee.value.email
  );
  closeConfirm();
};

const toggleBlock = (employee) => {
  employee.status = !employee.status;
};

const refresh = () => {
  searchEmail.value = "";
};
</script>

<style></style>
