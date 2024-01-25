<template>
    <div>
      <form>
        <label for="employee"><strong>Employee: </strong></label>
        <select name="employee" id="employee" @change="getEmployeeHierarchy($event)">
          <option value="">Please select an employee</option>
          <option v-for="employee in employees" :key="employee.id" :value="employee.name" >{{employee.name}}</option>
        </select>
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
      employees: [],
      employeeHierarchy: null,
      employeeName: []
    }
  },
  methods: {
    async getEmployeeList() {
      const res = await fetch('http://localhost:9090/api/v1/employees');
      const {data} = await res.json();
      this.employees = data;
    },

    async getEmployeeHierarchy(event) {
      this.employeeName = [];
      const res = await fetch("http://localhost:9090/api/v1/employees/search?name="+event.target.value);
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
