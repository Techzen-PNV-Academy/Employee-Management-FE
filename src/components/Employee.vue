<template>
  <div>
    <!-- Existing Table Section -->
    <h1 class="text-2xl font-bold mb-4">Danh sách nhân viên</h1>
    <button
      @click="fetchEmployees"
      class="bg-blue-500 text-white py-2 px-4 rounded"
    >
      Load Employees
    </button>

    <table class="table-auto w-full mt-4 border-collapse border">
      <thead>
        <tr class="bg-gray-200">
          <th class="border px-4 py-2">STT</th>
          <th class="border px-4 py-2">Tên</th>
          <th class="border px-4 py-2">Ngày sinh</th>
          <th class="border px-4 py-2">Giới tính</th>
          <th class="border px-4 py-2">Lương</th>
          <th class="border px-4 py-2">SĐT</th>
          <th class="border px-4 py-2">DEPARTERMENT</th>
          <th class="border px-4 py-2">Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(employee, index) in employees" :key="employee.id">
          <td class="border px-4 py-2">{{ index + 1 }}</td>
          <td class="border px-4 py-2">{{ employee.name }}</td>
          <td class="border px-4 py-2">{{ employee.dob }}</td>
          <td class="border px-4 py-2">{{ employee.gender }}</td>
          <td class="border px-4 py-2">
            {{ formatSalary(employee.salary) }} ₫
          </td>
          <td class="border px-4 py-2">{{ employee.phone }}</td>
          <td class="border px-4 py-2">{{ employee.selectedDepartment }}</td>
          <td class="border px-4 py-2">
            <button
              @click="openEditModal(employee)"
              class="bg-yellow-500 text-white py-1 px-2 rounded mr-1"
            >
              Update
            </button>
            <button
              @click="deleteEmployee(employee)"
              class="bg-red-500 text-white py-1 px-2 rounded"
            >
              Delete
            </button>
            <button
              @click="viewDetail(employee)"
              class="bg-blue-500 text-white py-1 px-2 rounded"
            >
              View Detail
            </button>
          </td>
        </tr>
      </tbody>
    </table>

    <div
      v-if="showDetailModal && selectedEmployee"
      class="fixed inset-0 bg-gray-800 bg-opacity-75 flex justify-center items-center"
    >
      <div class="bg-white p-6 rounded-lg w-1/3">
        <h2 class="text-xl font-bold mb-4">Chỉnh sửa thông tin nhân viên</h2>

        <form @submit.prevent="saveEmployeeChanges">
          <div class="mb-4">
            <label class="block font-medium">Tên</label>
            <input
              v-model="selectedEmployee.name"
              type="text"
              class="w-full border p-2 rounded"
            />
          </div>

          <div class="mb-4">
            <label class="block font-medium">Ngày sinh</label>
            <input
              v-model="selectedEmployee.dob"
              type="date"
              class="w-full border p-2 rounded"
            />
          </div>

          <div class="mb-4">
            <label class="block font-medium">Giới tính</label>
            <select
              v-model="selectedEmployee.gender"
              class="w-full border p-2 rounded"
            >
              <option value="MALE">Nam</option>
              <option value="FEMALE">Nữ</option>
            </select>
          </div>

          <div class="mb-4">
            <label class="block font-medium">Lương</label>
            <input
              v-model.number="selectedEmployee.salary"
              type="number"
              class="w-full border p-2 rounded"
              placeholder="Nhập lương"
            />
          </div>

          <div class="mb-4">
            <label class="block font-medium">Số điện thoại</label>
            <input
              v-model="selectedEmployee.phone"
              type="text"
              class="w-full border p-2 rounded"
              placeholder="Nhập số điện thoại"
            />
          </div>

          <div class="mb-4">
            <label class="block font-medium">Bộ phận</label>
            <select
              v-model="selectedEmployee.selectedDepartment"
              class="w-full border p-2 rounded"
            >
              <option value="1">Quản lí</option>
              <option value="2">Kế toán</option>
              <option value="3">Sản xuất</option>
              <option value="4">Sale</option>
            </select>
          </div>

          <div class="flex justify-end gap-2 mt-4">
            <button
              @click="closeDetailModal"
              type="button"
              class="bg-gray-500 text-white px-4 py-2 rounded"
            >
              Close
            </button>
            <button
              type="submit"
              class="bg-blue-500 text-white px-4 py-2 rounded"
            >
              Save
            </button>
          </div>
        </form>
      </div>
    </div>

    <div class="bg-gray-100 mt-8 p-4 rounded-lg shadow">
      <h2 class="text-lg font-bold mb-4">Tìm kiếm nhân viên</h2>
      <form @submit.prevent="searchEmployees" class="grid grid-cols-3 gap-4">
        <div>
          <label class="block mb-1 text-gray-700"
            >Tên (Tìm kiếm gần đúng)</label
          >
          <input
            v-model="searchForm.name"
            type="text"
            class="w-full border rounded p-2"
            placeholder="Nhập tên"
          />
        </div>

        <div>
          <label class="block mb-1 text-gray-700">Ngày sinh từ</label>
          <input
            v-model="searchForm.dobFrom"
            type="date"
            class="w-full border rounded p-2"
          />
        </div>

        <div>
          <label class="block mb-1 text-gray-700">Ngày sinh đến</label>
          <input
            v-model="searchForm.dobTo"
            type="date"
            class="w-full border rounded p-2"
          />
        </div>

        <div>
          <label class="block mb-1 text-gray-700">Giới tính</label>
          <select v-model="searchForm.gender" class="w-full border rounded p-2">
            <option value="">Tất cả</option>
            <option value="MALE">Nam</option>
            <option value="FEMALE">Nữ</option>
          </select>
        </div>

        <div>
          <label class="block mb-1 text-gray-700">Mức lương</label>
          <select
            v-model="searchForm.salaryRange"
            class="w-full border rounded p-2"
          >
            <option value="">Tất cả</option>
            <option value="0-1000">Dưới 1000</option>
            <option value="1000-2000">1000 - 2000</option>
            <option value="2000+">Trên 2000</option>
          </select>
        </div>

        <div>
          <label class="block mb-1 text-gray-700"
            >Số điện thoại (Tìm kiếm gần đúng)</label
          >
          <input
            v-model="searchForm.phone"
            type="text"
            class="w-full border rounded p-2"
            placeholder="Nhập số điện thoại"
          />
        </div>

        <div>
          <label class="block mb-1 text-gray-700">Tìm bộ phận</label>
          <select
            v-model="searchForm.selectedDepartment"
            class="w-full border rounded p-2"
          >
            <option value="0">Tất cả</option>
            <option value="1">Quản lí</option>
            <option value="2">Kế toán</option>
            <option value="3">Sản xuất</option>
            <option value="4">Sale</option>
          </select>
        </div>

        <div class="col-span-3 flex justify-end gap-2 mt-4">
          <button
            type="button"
            @click="resetSearch"
            class="bg-gray-500 text-white px-4 py-2 rounded"
          >
            Đặt lại
          </button>
          <button
            type="submit"
            class="bg-blue-500 text-white px-4 py-2 rounded"
          >
            Tìm kiếm
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
import { getEmployees, deleteEmployee } from "../services/employeeService";
import axios from "axios";

export default {
  data() {
    return {
      employees: [],
      departments: [],
      searchForm: {
        name: "",
        dobFrom: "",
        dobTo: "",
        gender: "",
        salaryRange: "",
        phone: "",
        selectedDepartment: null,
      },
      showDetailModal: false,
      selectedEmployee: null,
    };
  },
  methods: {
    openEditModal(employee) {
      this.selectedEmployee = { ...employee };
      this.showDetailModal = true;

      if (this.selectedEmployee.dob) {
        this.selectedEmployee.dob = this.formatDateToInput(
          this.selectedEmployee.dob
        );
      }
      this.showDetailModal = true;
    },

    formatDateToInput(dateString) {
      const date = new Date(dateString);
      const year = date.getFullYear();
      const month = String(date.getMonth() + 1).padStart(2, "0"); // Ensure two digits
      const day = String(date.getDate()).padStart(2, "0");
      return `${year}-${month}-${day}`;
    },

    closeDetailModal() {
      this.showDetailModal = false;
      this.selectedEmployee = null;
    },

    viewDetail(employee) {
      this.selectedEmployee = employee;
      this.showDetailModal = true;
    },

    async fetchEmployees() {
      try {
        const response = await axios.get("http://localhost:8080/employees");
        this.employees = response.data.data;
      } catch (error) {
        console.error("Failed to fetch employees:", error);
      }
    },

     async searchEmployees() {
      try {
        // Log search criteria for debugging
        console.log("Search criteria:", this.searchForm);

        // Fetch employees from the backend with the search criteria
        const response = await axios.get(
          'http://localhost:8080/employees',
          this.searchForm
        );
        this.employees = response.data; // Update the employees list with the search result
      } catch (error) {
        console.error('Error fetching employees:', error);
      }
    },

    resetSearch() {
      this.searchForm = {
        name: "",
        dobFrom: "",
        dobTo: "",
        gender: "",
        salaryRange: "",
        phone: "",
        departments: "",
      };
      this.fetchEmployees();
    },

    async saveEmployeeChanges() {
      console.log(this.selectedEmployee.id);
      try {
        await axios.put(
          `http://localhost:8080/employees`,
          this.selectedEmployee
        );
        this.fetchEmployees();

        alert("Cập nhật thành công!");
        this.closeDetailModal();
      } catch (error) {
        console.error("Failed to save employee changes:", error);
        alert("Cập nhật thất bại!");
      }
    },

    async deleteEmployee(employee) {
      const id = employee.id;
      try {
        await axios.delete(`http://localhost:8080/employees/${id}`);
        alert("Xóa thành công!");
        this.employees = this.employees.filter((emp) => emp.id !== id);
      } catch (error) {
        console.error("Failed to delete employee:", error);
        alert("Xóa thất bại!");
      }
    },

    async fetchDepartments() {
      try {
        const response = await axios.get('http://localhost:8080/department');
        this.departments = response.data;
      } catch (error) {
        console.error('Failed to fetch departments:', error);
      }
    },

    formatSalary(salary) {
      return salary.toLocaleString();
    },
  },
  mounted() {
    this.fetchEmployees();
    this.fetchDepartments();
  },
};
</script>

<style scoped></style>
