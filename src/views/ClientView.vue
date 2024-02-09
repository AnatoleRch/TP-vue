<template>
    <main>
        <div>
            <h1>Les clients</h1>
        </div>
        <div>
            <table>
                <caption>Liste des Clients</caption>
                <tr>
                    <th>Code</th>
                    <th>Société</th>
                    <th>Contact</th>
                    <th>Ville</th>
                </tr>
                <!-- Si le tableau des Clients est vide -->
                <tr v-if="data.listeClients.length === 0">
                    <td colspan="4">Veuillez patienter, chargement des Clients...</td>
                </tr>
                <!-- Si le tableau des Clients n'est pas vide -->
                <tr v-for="client in data.listeClients" :key="client.code">
                    <td>{{ client.code }}</td>
                    <td>{{ client.societe }}</td>
                    <td>{{ client.contact }}</td>
                    <td>{{ client.adresse.ville }}</td>
                </tr>
                <tr>
                    <th>
                        <button @click="chargeDebut(0)">
                            debut
                        </button></th>
                    <th>
                        <button @click="chargePrecedant()">
                            ←
                        </button></th>
                    <th>
                        <button @click="chargeSuivant()">
                            →
                        </button></th>
                    <th>
                        <button @click="chargeFin()">
                            fin
                        </button></th>
                </tr>
            </table>
        </div>
    </main>
</template>

<script setup>
import { reactive, onMounted } from "vue";
import { doAjaxRequest } from "@/api";
let page = 0;
let totalpage=0
// Pour réinitialiser le formulaire
const clientVide = {
    libelle: "",
    description: ""
};

let data = reactive({
    // Les données saisies dans le formulaire
    formulaireClient: { ...clientVide },
    // La liste des Clients affichée sous forme de table
    listeClients: []
});

function showError(error) {
    console.log("Erreur : status %d", error.status)
    console.log(error.body);
    alert(error.message);
}

function chargeClients() {
    // Appel à l'API pour avoir la liste des Clients
    // Trié par code, descendant
    // Verbe HTTP GET par défaut
    doAjaxRequest("/api/clients?size=5&page=0")
        .then((json) => {
            console.log(json)
            data.listeClients = json._embedded.clients;
            totalpage=json.page.totalPages
            page = json.page.number

        })
        .catch(showError);
}
//json.page.number
function chargeFin() {
    doAjaxRequest("/api/clients?size=5&page="+totalpage)
        .then((json) => {
            console.log(json)
            data.listeClients = json._embedded.clients;
            page = json.page.number

        })
        .catch(showError);
}
function chargeDebut() {
    doAjaxRequest("/api/clients?size=5&page=0")
        .then((json) => {
            console.log(json)
            data.listeClients = json._embedded.clients;
            page = json.page.number

        })
        .catch(showError);
}
function chargeSuivant() {
    page=page+1
    doAjaxRequest("/api/clients?size=5&page="+page)
        .then((json) => {
            console.log(json)
            data.listeClients = json._embedded.clients;
            page = json.page.number
        })
        .catch(showError);
}
function chargePrecedant() {
    page=page-1
    doAjaxRequest("/api/clients?size=5&page="+page)
        .then((json) => {
            console.log(json)
            data.listeClients = json._embedded.clients;
            page = json.page.number

        })
        .catch(showError);
}


function ajouteClient() {
    // Ajouter une Client avec les données du formulaire
    const options = {
        method: "POST", // Verbe HTTP POST pour ajouter un enregistrement
        body: JSON.stringify(data.formulaireClient),
        headers: {
            "Content-Type": "application/json",
            "Accept": "application/json"
        },
    };
    doAjaxRequest("/api/clients", options)
        .then(() => {
            // Réinitialiser le formulaire
            data.formulaireClient = { ...clientVide };
            // Recharger la liste des Clients
            chargeClients();
        })
        .catch(showError);
}
/**
 * Supprime une entité
 * @param entityRef l'URI de l'entité à supprimer
 */
function deleteEntity(entityRef) {
    doAjaxRequest(entityRef, { method: "DELETE", headers: { "Accept": "application/json" }})
        .then(chargeClients)
        .catch(showError);
}

// A l'affichage du composant, on affiche la liste
onMounted(chargeClients);

</script>


<style scoped>
td,
th {
    border: 1px solid #ddd;
    padding: 8px;
}

th {
    padding-top: 12px;
    padding-bottom: 12px;
    text-align: left;
    background-color: #232623;
    color: rgb(255, 255, 255);
}
</style>
