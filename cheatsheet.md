`channel.constants.ts`
``` typescript
import { ChannelOptions } from '@aeternity/aepp-sdk/es/channel/internal';
import BigNumber from 'bignumber.js';
import { INITIATOR_KEYPAIR, RESPONDER_KEYPAIR } from '../sdk/sdk.constants';

export const WEBSOCKET_URL = 'ws://localhost:3014/channel';
export const MUTUAL_CHANNEL_CONFIGURATION: Omit<
  ChannelOptions,
  'role' | 'sign'
> & {
  minimumDepthStrategy: 'plain' | 'txFee';
  minimumDepth: number;
} = {
  url: WEBSOCKET_URL,
  pushAmount: 0,
  initiatorAmount: new BigNumber('5e18'),
  responderAmount: new BigNumber('5e18'),
  initiatorId: INITIATOR_KEYPAIR.publicKey,
  responderId: RESPONDER_KEYPAIR.publicKey,
  host: 'localhost',
  port: 3333,
  channelReserve: 2,
  // We're using 0 for lockPeriod in order to quickly close the channel
  // in cases where the initiator needs to solo close it and finally.
  // execute channel_settle transaction
  // read more: https://github.com/aeternity/protocol/blob/master/channels/ON-CHAIN.md#channel_settle
  lockPeriod: 0,
  // workaround: minimize the number of node hangs
  timeoutIdle: 2 * 60 * 60 * 1000,
  debug: true,
  // How to calculate minimum depth - either txfee (default) or plain. We use
  // `plain` with `minimumDepth` in order to reduce delay.
  minimumDepthStrategy: 'plain',
  minimumDepth: 0,
} as const;
```
----
`channel.types.ts`
``` typescript
import { Encoded } from '@aeternity/aepp-sdk/es/utils/encoder';

export interface Update {
  call_data: Encoded.ContractBytearray;
  contract_id: Encoded.ContractAddress;
  op: 'OffChainCallContract' | 'OffChainNewContract';
  code?: Encoded.ContractBytearray;
  owner?: Encoded.AccountAddress;
  caller_id?: Encoded.AccountAddress;
}
```
---
`contract.constants.ts`
``` typescript
import BigNumber from 'bignumber.js';

export const CONTRACT_CONFIGURATION = {
  deposit: 0e18,
  vmVersion: 5,
  abiVersion: 3,
} as const;

export const CONTRACT_NAME = 'RockPaperScissors';
export const GAME_STAKE = new BigNumber('0.01e18');
export enum Moves {
  rock = 'rock',
  paper = 'paper',
  scissors = 'scissors',
}

export enum Methods {
  init = 'init',
  provide_hash = 'provide_hash',
  get_state = 'get_state',
  player1_move = 'player1_move',
  reveal = 'reveal',
  player1_dispute_no_reveal = 'player1_dispute_no_reveal',
  player0_dispute_no_move = 'player0_dispute_no_move',
  set_timestamp = 'set_timestamp',
}
```
---
`game-round.ts`
``` typescript
import { Encoded } from '@aeternity/aepp-sdk/es/utils/encoder';
import BigNumber from 'bignumber.js';
import { defineStore } from 'pinia';
import { Moves } from '../utils/contract/contract.constants';

interface GameRoundStore {
  initiatorMove?: Moves;
  responderMove?: Moves;
  hasRevealed: boolean;
  index: number;
  isComplete: boolean;
  winner?: Encoded.AccountAddress;
  hashKey: string;
  initiatorBalance?: BigNumber;
  responderBalance?: BigNumber;
}

export const useGameRoundStore = defineStore<
  'gameRound',
  GameRoundStore,
  // eslint-disable-next-line @typescript-eslint/ban-types
  {},
  { startNewRound: () => void; }
>('gameRound', {
  state: () => ({
    initiatorMove: undefined,
    responderMove: undefined,
    hasRevealed: false,
    index: 0,
    isComplete: false,
    winner: undefined,
    hashKey: Math.random().toString(16).substring(2, 8),
    initiatorBalance: undefined,
    responderBalance: undefined,
  }),
  actions: {
    startNewRound() {
      this.index++;
      this.initiatorMove = undefined;
      this.responderMove = undefined;
      this.isComplete = false;
      this.hasRevealed = false;
      this.winner = undefined;
      this.hashKey = Math.random().toString(16).substring(2, 8);
    },
  },
});
```
---
`channel-status.vue`
``` typescript
<script setup lang="ts">
defineProps<{
  status: string;
}>();
</script>

<template>
  <div class="channel-status">Channel status: {{ status }}</div>
</template>

<style scoped lang="scss">
.channel-status {
  position: absolute;
  top: 10px;
  left: 10px;
  background-color: rgba(255, 255, 255, 0.87);
  color: #242424;
  border-radius: 5px;
  padding: 5px 10px;
  width: max-content;
}
</style>
```
----
`game-screen.vue`
```typescript
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
```
---
`player-controls.vue`
``` typescript
<style scoped lang="scss">
.player-controls {
  .moves {
    margin-top: 10px;
    display: grid;
    grid-template-rows: 1fr 1fr 1fr;
    gap: 10px;
  }
}
</style>
```

`empty-component.vue`
``` typescript
<script setup lang="ts">

</script>

<template>
  <div class="">
  </div>
</template>

<style scoped lang="scss"></style>
```