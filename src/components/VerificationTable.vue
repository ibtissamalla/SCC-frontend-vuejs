<template>
  <v-container>
    <v-row>
      <v-col cols="12">
        <v-card>
          <v-card-title>
            Tableau de Vérification des Anomalies
          </v-card-title>
          <v-card-subtitle>
            <v-btn @click="fetchMonthlyAnomalies" color="primary">Rafraîchir</v-btn>
          </v-card-subtitle>
          <v-data-table
            :headers="headers"
            :items="anomalies"
            item-key="month"
          >
            <template v-slot:default="{ items }">
              <thead>
                <tr>
                  <th v-for="header in headers" :key="header.value">{{ header.text }}</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="item in items" :key="item.month">
                  <td>{{ item.month }}</td>
                  <td>{{ item.anomalies_count }}</td>
                  <td>
                    <v-btn @click="viewMonthlyConsumption(item.month.split('/')[0])" color="primary">Voir Consommations</v-btn>
                  </td>
                </tr>
              </tbody>
            </template>
          </v-data-table>
        </v-card>
      </v-col>
    </v-row>

    <!-- Dialogue pour afficher la consommation mensuelle -->
    <v-dialog v-model="dialog" max-width="1200px">
      <v-card>
        <v-card-title>
          Consommations pour le mois {{ selectedMonth }}
        </v-card-title>
        <v-card-subtitle>
          <v-btn @click="downloadExcel" color="primary">Télécharger en Excel</v-btn>
          <v-btn @click="dialog = false" color="primary">Fermer</v-btn>
        </v-card-subtitle>
        <v-card-text>
          <v-data-table
            :headers="consumptionHeaders"
            :items="consumption"
            item-key="id"
          >
            <template v-slot:default="{ items }">
              <thead>
                <tr>
                  <th v-for="header in consumptionHeaders" :key="header.value">{{ header.text }}</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="item in items" :key="item.id">
                  <td>{{ item.matricule }}</td>
                  <td>{{ item.nomPrenom }}</td>
                  <td>{{ item.date }}</td>
                  <td>{{ item.heure }}</td>
                  <td>{{ item.description }}</td>
                  <td>{{ item.quantite }}</td>
                  <td>{{ item.mt_sub }}</td>
                  <td>{{ item.dateControle }}</td>
                  <td>{{ item.anomalie }}</td>
                </tr>
              </tbody>
            </template>
          </v-data-table>
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-container>
</template>


<script>
import axios from 'axios';
import * as XLSX from 'xlsx'; // Importer la bibliothèque xlsx

export default {
  data() {
    return {
      headers: [
        { text: 'Mois', value: 'month' },
        { text: 'Nombre d\'Anomalies', value: 'anomalies_count' },
        { text: 'Actions', value: 'actions', sortable: false }
      ],
      consumptionHeaders: [
        { text: 'Matricule', value: 'matricule' },
        { text: 'Nom & Prénom', value: 'nomPrenom' },
        { text: 'Date', value: 'date' },
        { text: 'Heure', value: 'heure' },
        { text: 'Description', value: 'description' },
        { text: 'Quantité', value: 'quantite' },
        { text: 'Montant', value: 'mt_sub' },
        { text: 'Date de Contrôle', value: 'dateControle' },
        { text: 'Anomalie', value: 'anomalie' }
      ],
      anomalies: [],
      consumption: [],
      dialog: false,
      selectedMonth: ''
    };
  },
  methods: {
   async fetchMonthlyAnomalies() {
    try {
      const year = new Date().getFullYear(); // Utilisez l'année en cours ou toute autre logique pour obtenir l'année
      const months = Array.from({ length: 12 }, (_, i) => i + 1);
      const anomalyRequests = months.map(month =>
        axios.get(`http://127.0.0.1:8000/api/consommations/anomalies/${year}/${month}`)
      );
      const responses = await Promise.all(anomalyRequests);
      this.anomalies = responses.map((response, index) => ({
        month: `${index + 1}/${year}`, // Formater le mois/année
        anomalies_count: response.data.anomalies_count
      }));
    } catch (error) {
      console.error('Erreur lors de la récupération des anomalies mensuelles:', error.message);
      // Afficher un message d'erreur dans l'interface utilisateur si nécessaire
    }
  },
    async viewMonthlyConsumption(month) {
      try {
        this.selectedMonth = month;
        const response = await axios.get(`http://127.0.0.1:8000/api/consommations/month/${month}`);
        this.consumption = response.data;
        this.dialog = true;
      } catch (error) {
        console.error('Erreur lors de la récupération des consommations mensuelles:', error.message);
        // Afficher un message d'erreur dans l'interface utilisateur si nécessaire
      }
    },
    downloadExcel() {
      // Convertir les données en format Excel
      const ws = XLSX.utils.json_to_sheet(this.consumption, {
        header: this.consumptionHeaders.map(header => header.value)
      });
      const wb = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(wb, ws, `Consommations-${this.selectedMonth}`);
      
      // Générer le fichier Excel et le télécharger
      XLSX.writeFile(wb, `Consommations-${this.selectedMonth}.xlsx`);
    }
  },
  mounted() {
    this.fetchMonthlyAnomalies();
  }
};
</script>
<style scoped>
.v-card {
  margin-bottom: 20px;
}
</style>
