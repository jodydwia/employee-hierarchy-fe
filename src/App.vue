<template>
    <div>
      <form class="employee__form">
          <div>
            <label for="selectEmployee"><strong>Select Employee: </strong></label>
            <select name="selectEmployee" id="selectEmployee" ref="selectEmployee" @change="getEmployeeHierarchy($event).then(()=>{
                  this.$refs.searchEmployee.value='';
          })">
              <option value="">Please select an employee</option>
              <option v-for="employee in employees" :key="employee.id" :value="employee.name" >{{employee.name}}</option>
            </select>
          </div>
        <div>
          <label for="selectEmployee"><strong>Search Employee: </strong></label>
          <input type="text" name="searchEmployee" id="searchEmployee" ref="searchEmployee" @keydown.enter="getEmployeeHierarchy($event).then(()=>{
                  this.$refs.selectEmployee.value='';
          })" placeholder="type name then enter">
        </div>
      </form>
    </div>
    <div class="employee__hierarchy">
      <p v-show="employeeHierarchy"><strong>Employee Hierarchy:</strong></p>
      <p>{{employeeName.length > 0 ? employeeName.join(" has manager --> ") : employeeHierarchy?.message}}</p>
      <hr>
      <p v-show="employeeHierarchy"><strong>Json data:</strong></p>
      <pre>
        {{employeeHierarchy}}
      </pre>
    </div>
</template>

<script>

export default {
  name: 'EmployeeHierarchyApp',
  components: {
  },
  created() {
    this.getEmployeeList();
  },
  data() {
    return {
      API_URL: "http://localhost:9090/api/v1/employees",
      employees: [],
      employeeHierarchy: null,
      employeeName: []
    }
  },
  methods: {
    async getEmployeeList() {
      const res = await fetch(this.API_URL);
      const {data} = await res.json();
      this.employees = data;
    },

    async getEmployeeHierarchy(event) {
      event.preventDefault();
      this.employeeName = [];
      const res = await fetch(`${this.API_URL}/search?name=${event.target.value.toLowerCase()}`);
      const {data} = await res.json();
      this.employeeHierarchy = data;
      this.getEmployeeName(data);
    },

    getEmployeeName(data) {
      if(data.value) {
        if(this.employeeName.length === 0) {
          this.employeeName.push(data.value.name)
        }
        if(data.childNodes.length > 0) {
          const manager = data.childNodes[0];
          this.employeeName.push(manager.value.name);
          this.getEmployeeName(manager);
        }
      }
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin: 60px auto;
  max-width: 80%;
  text-align: center;
  border: 1px solid black;
  border-radius: 8px;
  padding: 10px;
}


.employee__form {
  display: flex;
  justify-content: center;
  gap: 25px;
}
.employee__hierarchy {
  max-width: 50%;
  margin: 10px auto;
  text-align: left;
  padding: 5px;
  background-color: lightgrey;
  border: 1px dashed black;
  border-radius: 5px;
}
</style>
