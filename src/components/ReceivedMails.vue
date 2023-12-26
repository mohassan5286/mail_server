<template>
  <div class="pageReceivedMails">
    <nav class="navbar">
      <div class="container-fluid">
        <form class="d-flex" role="search" @submit.prevent="deleteSelectedItems">
          <input v-model = "searchInput" class="form-control me-2" type="search" placeholder="Search" aria-label="Search">
          <button class="btn btn-outline-success" type="submit" @click = "search">Search</button>

          <div class="dropdown" id="searchBy">
            <button type="button" class="btn btn-outline-primary dropdown-toggle" data-bs-toggle="dropdown" aria-expanded="false" id = "search-button">{{ searchByText }}</button>
            <ul class="dropdown-menu">
              <li @click="updateSearchBy('Date')" class="dropdown-item">Date</li>
              <li @click="updateSearchBy('Sender')" class="dropdown-item">Sender</li>
              <li @click="updateSearchBy('Subject')" class="dropdown-item">Subject</li>
              <li @click="updateSearchBy('Body')" class="dropdown-item">Body</li>
              <li @click="updateSearchBy('Importance')" class="dropdown-item">Importance</li>
            </ul>
          </div>

          <input v-model = "filterInput" class="form-control me-2" type="search" placeholder="Filter" aria-label="Search">
          <button class="btn btn-outline-warning" type="submit" @click = "filter">Filter</button>

          <div class="dropdown" id="filterBy">
            <button type="button" class="btn btn-outline-primary dropdown-toggle" data-bs-toggle="dropdown" aria-expanded="false">{{ filterByText }}</button>
            <ul class="dropdown-menu">
              <li @click="updateFilterBy('Date')" class="dropdown-item">Date</li>
              <li @click="updateFilterBy('Sender')" class="dropdown-item">Sender</li>
              <li @click="updateFilterBy('Subject')" class="dropdown-item">Subject</li>
              <li @click="updateFilterBy('Body')" class="dropdown-item">Body</li>
              <li @click="updateFilterBy('Importance')" class="dropdown-item">Importance</li>
            </ul>
          </div>

          <h3>Received</h3>

          <div class="dropdown" id="type">
            <button type="button" class="btn btn-outline-dark dropdown-toggle" data-bs-toggle="dropdown" aria-expanded="false">{{ typeText }}</button>
            <ul class="dropdown-menu">
              <li @click="updateType('Default')" class="dropdown-item">Default</li>
              <li @click="updateType('Priority')" class="dropdown-item">Priority</li>
            </ul>
          </div>

          <div class="dropdown" id="sortBy">
            <button class="btn btn-outline-secondary dropdown-toggle" type="button" data-bs-toggle="dropdown" aria-expanded="false">{{ sortByText }}</button>
            <ul class="dropdown-menu">
              <li @click="updateSortBy('Date')" class="dropdown-item">Date</li>
              <li @click="updateSortBy('Sender')" class="dropdown-item">Sender</li>
              <li @click="updateSortBy('Subject')" class="dropdown-item">Subject</li>
              <li @click="updateSortBy('Body')" class="dropdown-item">Body</li>
              <li @click="updateSortBy('Importance')" class="dropdown-item">Importance</li>
            </ul>
          </div>

          <button @click="reload" type="button" class="btn btn-outline-success" id="reload">Reload</button>
          <button @click="deleteSelectedItems" class="btn btn-outline-danger" :disabled="!(isDeleteButtonEnabled || isPriorityDeleteButtonEnabled)" id="delete">Delete</button>
        </form>
      </div>
    </nav>
    <div v-show="Default">
      <ul class="default-items">
        <li v-for="email in defaultList" :key="email.id">
          <div class="email-item">
            <div class="email-header clearfix">
              <div class="email-sender">
                <strong>Sender:</strong> {{ email.sender }}
              </div>
              <div class="email-date">
                <strong>Date:</strong> {{ email.date }}
              </div>
            </div>
            <div class="email-subject">
              <strong>Subject:</strong> {{ email.subject }}
            </div>
            <div class="email-body">
              <strong>Body:</strong> {{ email.body }}
            </div>
            <div class="email-checkbox-wrapper">
              <input type="checkbox" v-model="email.isSelected" class="email-checkbox">
            </div>
          </div>
        </li>
      </ul>
    </div>

    <div v-show="Priority">
      <ul class="default-items">
        <li v-for="email in priorityList" :key="email.id">
          <div class="email-item">
            <div class="email-header clearfix">
              <div class="email-sender">
                <strong>Sender:</strong> {{ email.sender }}
              </div>
              <div class="email-date">
                <strong>Date:</strong> {{ email.date }}
              </div>
            </div>
            <div class="email-subject">
              <strong>Subject:</strong> {{ email.subject }}
            </div>
            <div class="email-body">
              <strong>Body:</strong> {{ email.body }}
            </div>
            <div class="email-checkbox-wrapper">
              <input type="checkbox" v-model="email.isSelected" class="email-checkbox">
            </div>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      searchByText: 'By',
      sortByText: 'Sort by',
      filterByText: 'By',
      typeText: 'Default',
      Default: true,
      Priority: false,
      defaultList: [],
      priorityList: [],
      searchInput: '',
      filterInput: '',
    };
  },
  computed: {
    isDeleteButtonEnabled() {
      return this.defaultList.some(email => email.isSelected);
    },
    isPriorityDeleteButtonEnabled() {
      return this.priorityList.some(email => email.isSelected);
    },
  },
  methods: {
    updateSearchBy(value) {
      this.searchByText = value;
    },

    async search() {
      const value = this.searchInput;
      const by    = this.searchByText.toLocaleLowerCase();
      let type    = this.typeText === 'Default' ? 'default' : 'priority';

      const MailDTO = {
        sender: this.$root.sender,
        type: type,
        value: value,
        by: by,
      };

      fetch('http://localhost:8080/search', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(MailDTO),
      })
        .then(response => {
          if (!response.ok) {
            throw new Error(`HTTP error! Status: ${response.status}`);
          }
          return response.json();
        })
        .then(data => {
          if (this.typeText === 'Default') {
            this.defaultList = Array.from(data);
          } else {
            this.priorityList = Array.from(data);
          }
        })
        .catch(error => console.error('Error:', error.message));
    },

    async updateSortBy(value) {
      this.sortByText = value;
      if (value === "Date" && this.typeText === 'Default')
        this.defaultList.sort((a, b) => new Date(a.date) - new Date(b.date)) ;
      else if (value === "Date" && this.typeText === 'Priority')
        this.priorityList.sort((a, b) => new Date(a.date) - new Date(b.date)) ;
      
      else if (value === "Sender" && this.typeText === 'Default')
        this.defaultList.sort((a, b) => a.sender.localeCompare(b.sender));
      else if (value === "Sender" && this.typeText === 'Priority')
        this.priorityList.sort((a, b) => a.sender.localeCompare(b.sender));

      else if (value === "Subject" && this.typeText === 'Default')
        this.defaultList.sort((a, b) => a.subject.localeCompare(b.subject));
      else if (value === "Subject" && this.typeText === 'Priority')
        this.priorityList.sort((a, b) => a.subject.localeCompare(b.subject));

      else if (value === "Body" && this.typeText === 'Default')
        this.defaultList.sort((a, b) => a.body.localeCompare(b.body));
      else if (value === "Body" && this.typeText === 'Priority')
        this.priorityList.sort((a, b) => a.body.localeCompare(b.body));

      else if (value === "Importance" && this.typeText === 'Default')
        this.defaultList.sort((a, b) => a.importance.localeCompare(b.importance));
      else if (value === "Importance" && this.typeText === 'Priority')
        this.priorityList.sort((a, b) => a.importance.localeCompare(b.importance));
    },

    updateFilterBy(value) {
      this.filterByText = value;
    },

    async filter() {
      const value = this.filterInput;
      const by    = this.filterByText.toLocaleLowerCase();
      let type    = this.typeText === 'Default' ? 'default' : 'priority';

      const MailDTO = {
        sender: this.$root.sender,
        type: type,
        value: value,
        by: by,
      };

      fetch('http://localhost:8080/filter', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(MailDTO),
      })
       
    },

    updateType(value) {
      this.typeText = value;
      this.Default = value === 'Default';
      this.Priority = value === 'Priority';
    },
    async deleteSelectedItems() {
      const selectedDefaultItems = this.defaultList.filter(email => email.isSelected);
      const selectedPriorityItems = this.priorityList.filter(email => email.isSelected);
      const selectedItems = selectedDefaultItems.concat(selectedPriorityItems);
      
      for (const selectedItem of selectedItems) {
        const MailDTO = {
          id: selectedItem.id,
          date: selectedItem.date,
          sender: selectedItem.sender,
          receivers: selectedItem.receivers,
          importance: selectedItem.importance,
          subject: selectedItem.subject,
          body: selectedItem.body,
        };

        try {
          const response = await fetch('http://localhost:8080/deleteDefaultPriority', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(MailDTO),
          });

          if (!response.ok) {
            throw new Error('Network response was not ok');
          }
        } catch (error) {
          console.error('Error during fetch:', error);
        }
      }

      this.defaultList = this.defaultList.filter(email => !email.isSelected);
      this.priorityList = this.priorityList.filter(email => !email.isSelected);
    },
    async deleteSelectedPriorityItems() {
      const selectedItems = this.priorityList.filter(email => email.isSelected);

      for (const selectedItem of selectedItems) {
        const MailDTO = {
          id: selectedItem.id,
          date: selectedItem.date,
          sender: selectedItem.sender,
          receivers: selectedItem.receivers,
          importance: selectedItem.importance,
          subject: selectedItem.subject,
          body: selectedItem.body,
        };

        try {
          const response = await fetch('http://localhost:8080/deletePriority', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(MailDTO),
          });

          if (!response.ok) {
            throw new Error('Network response was not ok');
          }
        } catch (error) {
          console.error('Error during fetch:', error);
        }
      }

      this.priorityList = this.priorityList.filter(email => !email.isSelected);
    },
    reload() {
      const MailDTO1 = {
        sender : this.$root.sender,
        type   : "default",
      };

      fetch('http://localhost:8080/showMails', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(MailDTO1),
      })
        .then(response => response.ok ? response.json() : Promise.reject(`HTTP error! Status: ${response.status}`))
        .then(data => this.defaultList = Array.from(data))
        .catch(error => console.error('Error:', error.message));

      const MailDTO2 = {
        sender : this.$root.sender, 
        type   : "priority",
      };

      fetch('http://localhost:8080/showMails', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(MailDTO2),
      })
        .then(response => response.ok ? response.json() : Promise.reject(`HTTP error! Status: ${response.status}`))
        .then(data => this.priorityList = Array.from(data))
        .catch(error => console.error('Error:', error.message));
    },
  },
};
</script>

<style scoped>
  .pageReceivedMails {
    top: 15%;
    display: none;
    width: 70%;
    height: 80%;
    left: 300px;
    position: absolute;
    border: 3px solid black;
    border-radius: 30px;
    box-shadow: 10px 10px 10px 10px;
  }

  h3 {
    color: rgb(0, 0, 0);
    margin-left: 50px;
    font-family: 'Grand Hotel';
  }

  .navbar {
    border: 1px solid;
    border-radius: 30px 30px 0 0;
    border-bottom: 2px solid rgb(37, 37, 37);
    height: 100px;
  }

  #searchBy,
  #filterBy,
  #sortBy,
  #delete,
  #reload,
  #type {
    margin-left: 5px;
  }

  #sortBy {
    margin-left: 20px;
  }

  #delete,
  #reload {
    margin-left: 20px;
  }

  #type {
    margin-left: 50px;
  }

  #search-button {
    margin-right: 5px;
  }

  .email-sender {
    float: left;
  }

  .email-date {
    float: right;
    text-align: right;
  }

  .clearfix::after {
    content: "";
    clear: both;
    display: block;
  }

  .default-items {
    list-style: none;
    padding: 0;
    margin: 0;
  }

  .email-item {
    border: 1px solid #ddd;
    padding: 15px;
    margin-bottom: 10px;
    width: 95%;
    margin-left: 25px;
  }

  .email-sender,
  .email-date {
    font-weight: bold;
  }

  .email-subject,
  .email-body {
    margin-top: 5px;
  }

  .email-subject {
    margin-top: 10px;
    font-size: 1.1em;
  }

  .email-item {
    position: relative;
  }

  .email-checkbox-wrapper {
    position: absolute;
    bottom: 0;
    right: 0;
    margin: 5px;
  }

</style>
