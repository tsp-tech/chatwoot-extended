<script setup>
/* global axios */
import { ref, computed, onMounted } from 'vue';
import { useToggle } from '@vueuse/core';
import { useStoreGetters } from 'dashboard/composables/store';
import { useI18n } from 'dashboard/composables/useI18n';
import {  } from 'dashboard/composables/store';

import WootDropdownItem from 'shared/components/ui/dropdown/DropdownItem.vue';
import WootDropdownMenu from 'shared/components/ui/dropdown/DropdownMenu.vue';

const getters = useStoreGetters();
const { t } = useI18n();

const arrowDownButtonRef = ref(null);
const isLoading = ref(false);

const [showActionsDropdown, toggleDropdown] = useToggle();
const closeDropdown = () => toggleDropdown(false);
const openDropdown = () => toggleDropdown(true);

const inboxId = computed(() => currentChat.value.inbox_id);
const remoteJid = computed(() => currentChat.value.meta.sender.identifier);
const currentChat = computed(() => getters.getSelectedChat.value);

const getRemoteJid = () => {
  try {
    return remoteJid;
  } catch {
    alert('Um erro ocorreu ao tentar obter o ID desta pessoa. Por favor, tente novamente mais tarde.');
    return null
  }
}

const getTypebotInfo = async (apikey) => {
  const options = { method: 'GET', headers: { apikey } };

  try {
    const res = await fetch('http://localhost:8080/typebot/find/TesteWpp', options)
    const data = await res.json();

    if (!data) throw res;

    return data;
  } catch (e) {
    console.log(e);
  }
};

const changeTypebotStatus = async (remoteJid, botId, apikey, newStatus) => {
  const status = newStatus || 'paused';

  const options = {
    method: 'POST',
    headers: {apikey, 'Content-Type': 'application/json'},
    body: `{"remoteJid":"${remoteJid}","status":"${status}"}`
  };

  try {
    console.log(await getTypebotInfo(apikey));

    const res = await fetch('http://localhost:8080/typebot/changeStatus/TesteWpp', options);

    const data = await res.json();
    if (!data) throw res;
    
    alert(`Bot "${data.typebot.typebot.typebot}" atualizado com sucesso\n\nNovo status: "${status}"\n\nInstância no EvolutionAPI: "${data.typebot.instanceName}."`)
  } catch (e) {
    console.error(e);
    alert('Um erro ocorreu. Pressione F12 e selecione a aba "Console" para ver mais informações')
  }
}

const getConversationParams = () => {
  const allConversations = document.querySelectorAll(
    '.conversations-list .conversation'
  );

  const activeConversation = document.querySelector(
    'div.conversations-list div.conversation.active'
  );
  const activeConversationIndex = [...allConversations].indexOf(
    activeConversation
  );
  const lastConversationIndex = allConversations.length - 1;

  return {
    all: allConversations,
    activeIndex: activeConversationIndex,
    lastIndex: lastConversationIndex,
  };
};

const changeBot = async (bot) => {
  console.log(currentChat.value, bot);
  const options = {
    method: 'POST',
    headers: {'Content-Type': 'application/json'},
    body: `{"botId":"${bot?.id}","inboxId":${inboxId.value},"instanceName":"${bot?.instanceName}"}`
  };

  try {
    await fetch(`http://localhost:8083/sessions/${remoteJid.value}/bot`, options);
  } catch (e) {
    alert('Um erro ocorreu');
    console.error(e);
  }
};

const toogleBot = (botId) => {
  closeDropdown();

  console.log(currentChat.value);
  if (isNaN(Number(botId))) return;

  isLoading.value = true;

  const remoteJid = getRemoteJid();
  if (!remoteJid) return;

  const apikey = prompt('Insira a chave da API do EvolutionAPI:');

  changeTypebotStatus(remoteJid, botId, apikey);
  isLoading.value = false;
};

const bots = ref([]);

const fetchData = async () => {
  try {
    const response = await fetch(`http://localhost:8083/bots`);
    const result = await response.json();

    if (!Array.isArray(result)) return;

    bots.value = result;
  } catch (error) {
    console.error('Error fetching data:', {error, remoteJid});
  }
};

onMounted(() => {
  fetchData();
});

</script>

<template>
  <div class="relative flex items-center justify-end resolve-actions">
    <div class="button-group">
      <woot-button
        ref="arrowDownButtonRef"
        color-scheme="primary"
        :disabled="isLoading"
        icon="chevron-down"
        emoji="🔽"
        @click="openDropdown"
      >
        Controlar bots
      </woot-button>
    </div>
    <div
      v-if="showActionsDropdown"
      v-on-clickaway="closeDropdown"
      class="dropdown-pane dropdown-pane--open left-auto top-[2.625rem] mt-0.5 right-0 max-w-[12.5rem] min-w-[9.75rem]"
    >
      <WootDropdownMenu class="mb-0">
        <WootDropdownItem>
          <woot-button
            variant="clear"
            color-scheme="secondary"
            size="small"
            icon="book-clock"
            @click="() => toogleBot(1)"
          >
            Pausar bot atual
          </woot-button>

          <hr style="margin: 4px 0;" />

          <span v-if="bots.length > 0" style="margin-top: 8px; padding-bottom: 4px; border-bottom: 1px solid #555555; display: block; width: 100%; text-align: center;">
            Ativar bot
          </span>

          <woot-button
            variant="clear"
            v-for="bot in bots"
            key="{{ bot.id }}"
            color-scheme="secondary"
            size="small"
            icon="book-clock"
            @click="() => changeBot(bot)"
          >
            {{ bot.name }}
          </woot-button>
        </WootDropdownItem>
      </WootDropdownMenu>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.dropdown-pane {
  .dropdown-menu__item {
    @apply mb-0;
  }
}
</style>
