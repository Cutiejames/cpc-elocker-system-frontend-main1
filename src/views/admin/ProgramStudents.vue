<template>
  <div class="container py-4">
    <button class="btn btn-outline-primary mb-3" @click="$router.back()">
      <i class="bi bi-arrow-left me-2"></i> Back
    </button>

    <h2 class="fw-bold text-primary mb-4">
      Students in {{ courseName }}
    </h2>

    <div class="table-container shadow-lg bg-white rounded-4 p-3">
      <table class="table table-hover align-middle text-center mb-0">
        <thead>
          <tr>
            <th>User ID</th>
            <th>Student ID</th>
            <th>Full Name</th>
            <th>Gender</th>
            <th>Email</th>
            <th>Role</th>
            <th>Status</th>
            <th>Actions</th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="student in students" :key="student.user_id">
            <!-- ✅ Show user_id instead of username -->
            <td>{{ student.user_id }}</td>

            <!-- Student ID -->
            <td>{{ student.stud_id }}</td>

            <!-- Full name (already concatenated in backend) -->
            <td>{{ student.full_name }}</td>

            <!-- Gender -->
            <td>{{ student.gender }}</td>

            <!-- Email -->
            <td>{{ student.email }}</td>

            <!-- Role -->
            <td>{{ student.role }}</td>

            <!-- Status -->
            <td>
              <span
                class="badge"
                :class="student.status === 'active' ? 'bg-success' : 'bg-secondary'"
              >
                {{ student.status || 'unknown' }}
              </span>
            </td>

            <!-- Actions -->
            <td>
              <div class="d-flex gap-2 justify-content-center">
                <button
                  class="btn btn-sm btn-warning"
                  :disabled="student.status === 'disabled'"
                  @click="disableAccount(student)"
                >
                  Disable
                </button>
                <button
                  class="btn btn-sm btn-success"
                  :disabled="student.status === 'active'"
                  @click="enableAccount(student)"
                >
                  Enable
                </button>
                <button
                  class="btn btn-sm btn-primary"
                  @click="openResetPasswordModal(student)"
                >
                  Reset Password
                </button>
              </div>
            </td>
          </tr>

          <!-- Empty state -->
          <tr v-if="students.length === 0">
            <td colspan="8" class="text-muted fst-italic py-4">
              No students found for this course
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Reset Password Modal -->
    <div class="modal fade" id="resetPasswordModal" tabindex="-1">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">
              Reset Password for User ID: {{ selectedStudent.user_id }}
            </h5>
            <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
          </div>
          <div class="modal-body">
            <input
              type="password"
              v-model="newPassword"
              class="form-control"
              placeholder="Enter new password"
            />
          </div>
          <div class="modal-footer">
            <button class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
            <button class="btn btn-primary" @click="submitResetPassword">
              Reset
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import bootstrap from "bootstrap/dist/js/bootstrap.bundle";

export default {
  name: "ProgramStudents",
  data() {
    return {
      students: [],
      courseId: null,
      courseName: "",
      selectedStudent: {},
      newPassword: "",
      resetPasswordModal: null,
    };
  },
  mounted() {
    this.courseId = this.$route.params.id;
    this.courseName = this.$route.query.name || "";
    this.fetchStudents();
  },
  methods: {
    // ✅ Fetch students from backend
    async fetchStudents() {
      try {
        const res = await axios.get(
          `http://localhost:3001/courses/${this.courseId}/students`,
          { params: { page: 1, limit: 50 } }
        );

        // Adjust depending on backend structure
        this.students = res.data.students || res.data;
      } catch (err) {
        console.error("Error fetching students:", err);
        alert("Failed to load students. Please try again later.");
      }
    },

    // ✅ Disable account
    async disableAccount(student) {
      if (!confirm(`Disable account for User ID ${student.user_id}?`)) return;

      try {
        const res = await axios.put(
          `http://localhost:3001/users/${student.user_id}/disable`
        );
        alert(res.data.message || "Account disabled successfully");
        this.fetchStudents();
      } catch (err) {
        console.error(err);
        alert(err.response?.data?.error || "Failed to disable account.");
      }
    },

    // ✅ Enable account
    async enableAccount(student) {
      if (!confirm(`Enable account for User ID ${student.user_id}?`)) return;

      try {
        const res = await axios.put(
          `http://localhost:3001/users/${student.user_id}/enable`
        );
        alert(res.data.message || "Account enabled successfully");
        this.fetchStudents();
      } catch (err) {
        console.error(err);
        alert(err.response?.data?.error || "Failed to enable account.");
      }
    },

    // ✅ Open reset password modal
    openResetPasswordModal(student) {
      this.selectedStudent = student;
      this.newPassword = "";
      const modalEl = document.getElementById("resetPasswordModal");
      this.resetPasswordModal = new bootstrap.Modal(modalEl);

      modalEl.addEventListener(
        "hidden.bs.modal",
        () => {
          this.newPassword = "";
        },
        { once: true }
      );

      this.resetPasswordModal.show();
    },

    // ✅ Submit reset password
    async submitResetPassword() {
      if (!this.newPassword) {
        alert("Password cannot be empty!");
        return;
      }

      if (this.newPassword.length < 6) {
        alert("Password must be at least 6 characters long");
        return;
      }

      try {
        const res = await axios.post("http://localhost:3001/reset-password", {
          user_id: this.selectedStudent.user_id,
          new_password: this.newPassword,
        });
        alert(res.data.message || "Password reset successfully!");
        this.resetPasswordModal.hide();
        this.fetchStudents();
      } catch (err) {
        console.error(err);
        alert(err.response?.data?.error || "Failed to reset password.");
      }
    },
  },
};
</script>

<style scoped>
.table-container {
  overflow-x: auto;
}
</style>
