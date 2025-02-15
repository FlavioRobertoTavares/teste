<script setup lang="ts">
import { defineProps, ref, onMounted } from 'vue';
import { useApiService } from '../services/apiService';
import router from '../router';

// Define as props esperadas
const props = defineProps<{
  id: string;
}>();
const { id } = props;

// Função para buscar a playlist por ID
const { getPlaylistByID, postSong, deleteSong, updatePlaylist, removePlaylist } = useApiService();

// Variáveis reativas para armazenar os detalhes da playlist
const name = ref('');
const description = ref('');
const newName = ref('');
const newDescription = ref('');
const song = ref('');
const categories = ref<string[]>([]);
const songs = ref<string[]>([]);
const showPopup = ref(false);
const showDetails = ref(false);

// Mostrar a playlist
onMounted(async () => {
  const response = await getPlaylistByID(id);
  if (response.code === 200) {
    name.value = response.data.name;
    description.value = response.data.description;
    categories.value = response.data.categories;
    songs.value = response.data.songs;
  } else {
    console.log('Erro ao buscar a playlist');
    console.log(response);
  }
});

// Função chamada ao clicar no botão "Adicionar música"
const add_Song = async () => {
  const response = await postSong({ id: id, song: song.value });
  if (response.code != 200) {
    console.log('Erro ao adicionar música');
    console.log(response);
  }
  router.go(0);
  showPopup.value = false;
};

// Função chamada ao clicar no botão "Excluir"
const remove_song = async (songName: string) => {
  const response = await deleteSong({ id: id, song: songName });
  if (response.code != 200) {
    console.log('Erro ao excluir música');
    console.log(response);
  }
  router.go(0);
};

// Função chamada ao clicar no botão "Atualizar detalhes da playlist"
const update_playlist = async () => {
  if(newName.value == ''){newName.value = name.value;}
  if(newDescription.value == ''){newDescription.value = description.value;}
  const response = await updatePlaylist({ id: id, name: newName.value, description: newDescription.value });
  if (response.code != 200) {
    console.log('Erro ao atualizar playlist');
    console.log(response);
  } else {
    router.go(0);
    showDetails.value = false;
  }
};

const remove_playlist = async () => {
  const response = await removePlaylist({ id: id });
  if (response.code != 200) {
    console.log('Erro ao excluir playlist');
    console.log(response);
  }
  router.push("/playlists");
};


</script>

<template>
  <div class="playlist-details">
    <h1>{{ name }}</h1>
    <p>Descrição: {{ description }}</p>
    <div class="categories-container">
      <p>Categorias: {{ categories.join(', ') }}</p>
      <div class="button-group">
        <button class="add-song-btn" @click="showPopup = true">Adicionar música</button>
        <button class="update-details-btn" @click="showDetails = true">Atualizar detalhes da playlist</button>
        <button class="delete-btn" @click="remove_playlist">Deletar a playlist</button>
      </div>
    </div>
    <p>Músicas:</p>
    <div class="songs-container">
      <div v-for="song in songs" :key="song" class="song-item">
        {{ song }}
        <button class="delete-song-btn" @click="remove_song(song)">Excluir</button>
      </div>
    </div>
  </div>

  <!-- Pop-up para adicionar música -->
  <div v-if="showPopup" class="popup-overlay">
    <div class="popup">
      <h2>Adicionar Música</h2>
      <label for="song-name">Nome da Música:</label>
      <input type="text" id="song-name" v-model="song" />
      <button @click="add_Song">Adicionar</button>
      <button @click="showPopup = false">Cancelar</button>
    </div>
  </div>

  <!-- Pop-up para atualizar playlist -->
  <div v-if="showDetails" class="popup-details-overlay">
    <div class="popup-details">
      <h2>Atualizar Detalhes da Playlist</h2>
      <label for="playlist-name">Nome da Playlist:</label>
      <input type="text" id="playlist-name" v-model="newName" :placeholder="name" />
      <label for="playlist-description">Descrição da Playlist:</label>
      <input type="text" id="playlist-description" v-model="newDescription" :placeholder="description" />
      <button @click="update_playlist">Salvar</button>
      <button @click="showDetails = false">Cancelar</button>
    </div>
  </div>
</template>

<style scoped>
.playlist-details {
  padding: 16px; /* Espaço à esquerda e entre os elementos */
}

.playlist-details h1 {
  margin-bottom: 16px; /* Espaço abaixo do título */
}

.playlist-details p {
  margin-bottom: 8px; /* Espaço entre os parágrafos */
}

.categories-container {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 8px; /* Espaço entre a área de categorias e o próximo elemento */
}

.button-group {
  display: flex;
  gap: 8px; /* Espaço entre os botões */
}

.add-song-btn, .update-details-btn, .delete-btn {
  background-color: grey; /* Cor de fundo do botão */
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.add-song-btn:hover, .update-details-btn:hover, .delete-btn:hover {
  background-color: darkgrey; /* Cor de fundo ao passar o mouse */
}

.songs-container {
  display: flex;
  flex-direction: column; /* Exibir as músicas em coluna */
}

.song-item {
  background-color: plum; /* Cor de fundo plum */
  padding: 8px;
  margin-bottom: 4px; /* Espaço entre as músicas */
  border-radius: 4px;
  display: flex;
  justify-content: space-between; /* Adiciona espaço entre a música e o botão */
  align-items: center; /* Alinha os itens no centro verticalmente */
}

.delete-song-btn {
  background-color: red; /* Cor de fundo do botão Excluir */
  color: white;
  border: none;
  padding: 4px 8px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
}

.delete-song-btn:hover {
  background-color: darkred; /* Cor de fundo ao passar o mouse */
}

.popup-overlay, .popup-details-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5); /* Sobreposição escura */
  display: flex;
  align-items: center;
  justify-content: center;
}

.popup, .popup-details {
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  width: 300px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.popup h2, .popup-details h2 {
  margin-top: 0;
}

.popup label, .popup-details label {
  display: block;
  margin-top: 10px;
}

.popup input, .popup-details input {
  width: 100%;
  padding: 8px;
  margin-top: 4px;
  border: 1px solid grey;
  border-radius: 4px;
}

.popup button, .popup-details button {
  margin-top: 10px;
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.popup button:first-of-type, .popup-details button:first-of-type {
  background-color: green; /* Cor verde para adicionar */
  color: white;
}

.popup button:first-of-type:hover, .popup-details button:first-of-type:hover {
  background-color: darkgreen; /* Cor verde escuro ao passar o mouse */
}

.popup button:last-of-type, .popup-details button:last-of-type {
  background-color: red; /* Cor vermelha para cancelar */
  color: white;
  margin-left: 10px;
}

.popup button:last-of-type:hover, .popup-details button:last-of-type:hover {
  background-color: darkred /* Cor vermelha escura ao passar o mouse */
}
</style>
