<script setup lang="ts">
import PlayerControls from './player-controls.vue';
import { useGameRoundStore } from '../stores/game-round';
import { computed, ref } from 'vue';
import {
  INITIATOR_KEYPAIR,
  RESPONDER_KEYPAIR,
} from '../utils/sdk/sdk.constants';
import { closeChannel } from '../utils/channel/channel.service';

const channelIsClosing = ref(false);

const gameRoundStore = useGameRoundStore();
const winner = computed(() => {
  if (gameRoundStore.isComplete) {
    if (gameRoundStore.winner === INITIATOR_KEYPAIR.publicKey)
      return 'The initiator wins!';
    else if (gameRoundStore.winner === RESPONDER_KEYPAIR.publicKey)
      return 'The responder wins!';
    else return "It's a draw!";
  } else return null;
});

const initiatorBalance = computed(
  () => gameRoundStore.initiatorBalance?.dividedBy(1e18).toFormat(2) ?? '0.00'
);
const responderBalance = computed(
  () => gameRoundStore.responderBalance?.dividedBy(1e18).toFormat(2) ?? '0.00'
);
</script>

<template>
  <div class="game-screen">
    <div>{{ winner }}</div>
    <div class="players">
      <PlayerControls
        player="initiator"
        :balance="initiatorBalance"
        :isDisabled="
          gameRoundStore.index <= 0 || !!gameRoundStore.initiatorMove
        "
      />
      <PlayerControls
        player="responder"
        :balance="responderBalance"
        :isDisabled="
          !gameRoundStore.initiatorMove || !!gameRoundStore.responderMove
        "
      />
    </div>
    <div class="buttons">
      <button
        v-if="gameRoundStore.isComplete"
        @click="gameRoundStore.startNewRound()"
        :disabled="channelIsClosing"
      >
        New Round
      </button>
      <button
        v-if="gameRoundStore.isComplete"
        @click="
          () => {
            channelIsClosing = true;
            closeChannel();
          }
        "
        :disabled="channelIsClosing"
      >
        Close Channel
      </button>
    </div>
  </div>
</template>

<style scoped lang="scss">
.game-screen {
  display: grid;
  grid-template-rows: auto 200px auto;
  justify-content: center;
  align-items: center;
  gap: 20px;
  .players {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 30px;
  }
  .buttons {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 30px;
  }
}
</style>
