<template>
  <div id="app">
    <modal name="guess-modal">
      <h2 v-if="loading"><img alt="Loading" src="../assets/loader.gif" /></h2>
      <div v-if="selected">
        <h1>{{ selected.name }}</h1>
        <p v-if="!selected.country.length">No Guess...</p>
        <ul>
          <li v-for="country in selected.country" v-bind:key="country.country_id">
            {{ lookup(country.country_id) }}
          </li>
        </ul>
      </div>
    </modal>
    <table id="users">
      <thead>
        <tr>
          <th @click="sort('name')">Name</th>
          <th @click="sort('email')">Breed</th>
          <th @click="sort('gender')">Gender</th>
          <th></th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="user in sortedUsers" v-bind:key="user.id">
          <td>{{ user.name }}</td>
          <td>{{ user.email }}</td>
          <td>{{ user.gender }}</td>
          <button class="btn" @click="guess(user.name)">guess</button>
        </tr>
      </tbody>
    </table>
    <p>
      <button v-if="currentPage > 1" class="btn prev" @click="prevPage">Previous</button>
      <button v-if="isNotLastPage" class="btn next" @click="nextPage">Next</button>
    </p>
  </div>
</template>

<script lang="ts">
import lookupCode from "country-code-lookup";
import { Component, Vue } from "vue-property-decorator";
import userData from "../utils/data";

interface UserIntf {
  id: number;
  name: string;
  gender: string;
  email: string;
}
interface Result {
  name: string;
  country: { country_id: string }[];
}

type SortTypes = "name" | "gender" | "email";

@Component
export default class Users extends Vue {
  private users: UserIntf[] = [];
  private currentSort: SortTypes = "name";
  private currentSortDir = "asc";
  private pageSize = 10;
  private currentPage = 1;
  private loading = false;
  private selected: Result | null = null;

  created() {
    this.users = userData;
  }
  get isNotLastPage() {
    return this.currentPage * this.pageSize < this.users.length;
  }
  get sortedUsers() {
    return this.users
      .sort((a, b) => {
        let modifier = 1;
        if (this.currentSortDir === "desc") modifier = -1;
        if (a[this.currentSort] < b[this.currentSort]) return -1 * modifier;
        if (a[this.currentSort] > b[this.currentSort]) return 1 * modifier;
        return 0;
      })
      .filter((row, index) => {
        const start = (this.currentPage - 1) * this.pageSize;
        const end = this.currentPage * this.pageSize;
        if (index >= start && index < end) return true;
      });
  }

  public sort(s: SortTypes) {
    if (s === this.currentSort) {
      this.currentSortDir = this.currentSortDir === "asc" ? "desc" : "asc";
    }
    this.currentSort = s;
  }
  public nextPage() {
    if (this.isNotLastPage) this.currentPage++;
  }
  public prevPage() {
    if (this.currentPage > 1) this.currentPage--;
  }
  public guess(name: string) {
    this.selected = null;
    this.show();
    this.loading = true;
    fetch(`https://api.nationalize.io/?name=${name}`)
      .then(res => res.json())
      .then(res => {
        this.selected = res;
        this.loading = false;
      })
      .catch(() => {
        this.loading = false;
      });
  }
  show() {
    this.$modal.show("guess-modal");
  }

  public lookup(code: string) {
    return lookupCode.byIso(code).country;
  }
}
</script>

<style scoped>
#users {
  font-family: Arial, Helvetica, sans-serif;
  border-collapse: collapse;
  width: 100%;
}

#users td,
#users th {
  border: 1px solid #ddd;
  padding: 8px;
}

#users tr:nth-child(even) {
  background-color: #f2f2f2;
}

#users tr:hover {
  background-color: #ddd;
}

#users th {
  padding-top: 12px;
  padding-bottom: 12px;
  text-align: left;
  background-color: #fc6435;
  color: white;
}

.btn {
  background-color: #fc6435; /* Green */
  border: none;
  color: white;
  padding: 15px 32px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  margin: 5px 10px;
}
.next {
  background-color: #4caf50;
}
.prev {
  background-color: #df2f2f;
}
</style>
