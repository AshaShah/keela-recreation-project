<template>
<div class="contact-table-container">
    <!-- Button to toggle ContactForm visibility -->
    <button v-if="!isCoordinator" type="button" class="add-button" @click="addContact">Add Contact</button>
    <p class="counter">{{ contacts.length }} Contacts</p> <!-- Display total contacts count -->

    <!-- ContactForm Modal Overlay -->
    <div v-if="showContactForm" class="modal-overlay">
        <div class="contact-form-modal">
            <!-- Pass contactData as a prop to ContactForm -->
            <ContactForm @closeForm="closeForm" :initialContact="contactData" />
        </div>
    </div>
    <div class="table-container">
        <table class="contact-table">
            <thead>
                <tr>
                    <th>Full Name</th>
                    <th>Email</th>
                    <th>Tags</th>
                    <th>Phone Number</th>
                    <th v-if="!isCoordinator">Actions</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="contact in contacts" :key="contact._id">
                    <td>{{ contact.name }}</td>
                    <td>{{ contact.email }}</td>
                    <td>
                        <span v-if="contact.tags && contact.tags.length">{{ contact.tags.map(tag => tag.tagName).join(', ') }}</span>
                    </td>
                    <td>{{ contact.phone }}</td>
                    <td v-if="!isCoordinator">
                        <button class="actionbutton" @click="deleteContact(contact)">
                            <img class="action-icon" src="/delete.png" alt="Delete" />
                        </button>
                        <button class="actionbutton" @click="editContact(contact)">
                            <img class="action-icon" src="/edit.png" alt="Edit" />
                        </button>
                    </td>
                </tr>
            </tbody>
        </table>
    </div>
</div>
</template>

<script>
import ContactForm from '../../ui/forms/ContactForm.vue';
import {
    ContactsCollection
} from '../../db/ContactsCollection';
import {
    Meteor
} from 'meteor/meteor';

export default {
    name: "contactTable",
    components: {
        ContactForm,
    },
    data() {
        return {
            showContactForm: false,
            contactData: null, // Added to store data null so that it execute else value

        };
    },
    methods: {

        addContact() {
            // Open ContactForm for adding a new contact
            this.showContactForm = !this.showContactForm;
            this.contactData = null;
        },
        toggleContactForm(contact = null) {
            // Toggle the form and set contactData if available
            this.showContactForm = !this.showContactForm;
            this.contactData = contact;
        },
        deleteContact(contact) {
            const confirmDelete = confirm('Are you sure you want to delete this contact?');
            if (confirmDelete) {
                Meteor.call('Contacts.delete', contact._id)
            }
        },
        editContact(contact) {
            // Open ContactForm in edit mode
            this.toggleContactForm(contact);
        },
        closeForm() {
            this.showContactForm = !this.showContactForm;
            this.contactData = null;
        },         
    },
    computed: {
    isCoordinator() {
      const loggedInUser = Meteor.user();
      return loggedInUser && loggedInUser.profile.orgRole === 'Coordinator';
    },
  },
    meteor: {
        $subscribe: {
            'contacts': [],
        },
        contacts() {
            return ContactsCollection.find().fetch();
        },
    },
};
</script>

<style scoped>
.add-button {
    background-color: #7745d6;
    border-color: #7745d6;
    color: #fff;
    border-radius: 0.2rem;
    font-size: .875rem;
    line-height: 1.5;
    padding: 0.25rem 0.5rem;
    margin-left: 1270px;
    margin-top: 20px;
}

.contact-table-container {
    position: relative;
    overflow: auto;
    max-height: 700px;
}

.table-container {
    overflow-y: auto;
    max-height: inherit;
}

.contact-table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 10px;
    table-layout: fixed;
}

.contact-table th,
.contact-table td {
    border: 1px solid #ddd;
    padding: 12px 15px;
    word-wrap: break-word;
}

.contact-table th {
    background-color: #f2f2f2;
    font-weight: bold;
    border-bottom: 1px solid #ddd;
    position: sticky;
    /* Keep the header fixed */
    top: 0;
    /* Align with the top of the container */
    z-index: 1;
    /* Ensure the header appears above the table body */
}

.contact-table tbody tr {
    border-bottom: 1px solid #dddddd;
    text-align: left;
}

.contact-table tbody tr:nth-of-type(even) {
    background-color: #ffffff;
}

.modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 2;
}

.contact-form-modal {
    background-color: white;
    border: 1px solid #ddd;
    border-radius: 5px;
    padding: 20px;
    z-index: 3;
}

.actionbutton {
    background: none;
    border: none;
    cursor: pointer;
    padding: 10px;

}

.action-icon {
    width: 25px;
    height: 25px;
}

.counter {
    margin-right: 1270px;
    font-weight: bold;
}
</style>
