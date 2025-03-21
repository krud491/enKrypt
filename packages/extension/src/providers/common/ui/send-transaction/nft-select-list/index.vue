<template>
  <div class="nft-select-list">
    <div class="nft-select-list__header">
      <h3>Select NFT to send</h3>
      <a class="nft-select-list__close" @click="emit('close', false)">
        <close-icon />
      </a>
    </div>

    <nft-select-list-search />

    <custom-scrollbar
      class="nft-select-list__scroll-area"
      :settings="scrollSettings({ suppressScrollX: true })"
    >
      <nft-select-list-item
        v-for="(item, index) in nftList"
        :key="index"
        :item="item"
        @select-nft="emit('selectNft', $event)"
      />
    </custom-scrollbar>
  </div>
</template>

<script setup lang="ts">
import CloseIcon from '@action/icons/common/close-icon.vue';
import NftSelectListItem from './components/nft-select-list-item.vue';
import CustomScrollbar from '@action/components/custom-scrollbar/index.vue';
import NftSelectListSearch from './components/nft-select-list-search.vue';
import scrollSettings from '@/libs/utils/scroll-settings';
import { computed, onMounted, PropType, ref, watch } from 'vue';
import { EvmNetwork } from '@/providers/ethereum/types/evm-network';
import {
  NFTCollection,
  NFTItem,
  NFTItemWithCollectionName,
  NFTType,
} from '@/types/nft';
import { BitcoinNetwork } from '@/providers/bitcoin/types/bitcoin-network';
import { SolanaNetwork } from '@/providers/solana/types/sol-network';

const props = defineProps({
  network: {
    type: Object as PropType<EvmNetwork | BitcoinNetwork | SolanaNetwork>,
    default: () => ({}),
  },
  address: {
    type: String,
    default: '',
  },
  selectedNft: {
    type: Object as PropType<NFTItem>,
    default: () => ({}),
  },
});

const emit = defineEmits<{
  (e: 'selectNft', data: NFTItemWithCollectionName): void;
  (e: 'close', val: boolean): void;
}>();

const nftCollections = ref<NFTCollection[]>([]);
const nftList = computed(() => {
  const allItems: NFTItemWithCollectionName[] = [];
  nftCollections.value.forEach(col => {
    col.items.forEach(item => {
      allItems.push({ ...item, ...{ collectionName: col.name } });
    });
  });
  return allItems;
});
const updateNFTList = () => {
  if (props.network.NFTHandler) {
    props.network.NFTHandler(props.network, props.address).then(collections => {
      nftCollections.value = collections;
      if (nftList.value.length) {
        for (const nft of nftList.value) {
          if (
            props.selectedNft.contract === nft.contract &&
            props.selectedNft.id === nft.id
          ) {
            emit('selectNft', nft);
            return;
          }
        }
        emit('selectNft', nftList.value[0]);
      } else
        emit('selectNft', {
          id: '',
          contract: '',
          image: '',
          name: 'No NFTs found',
          url: '',
          collectionName: '',
          type: NFTType.ERC721,
        });
    });
  }
};
watch(
  () => props.address,
  () => {
    updateNFTList();
  },
);
onMounted(() => {
  updateNFTList();
});
</script>

<style lang="less">
@import '@action/styles/theme.less';
@import '@action/styles/custom-scroll.less';

.nft-select-list {
  width: 100%;
  background: #ffffff;
  position: fixed;
  box-shadow:
    0px 3px 6px rgba(0, 0, 0, 0.039),
    0px 7px 24px rgba(0, 0, 0, 0.19);
  border-radius: 12px;
  width: 428px;
  height: 568px;
  left: 356px;
  top: 16px;
  z-index: 12;

  &__header {
    position: relative;
    padding: 14px 56px 14px 16px;

    h3 {
      font-style: normal;
      font-weight: 700;
      font-size: 20px;
      line-height: 28px;
      color: @primaryLabel;
      margin: 0;
    }
  }

  &__close {
    position: absolute;
    top: 8px;
    right: 8px;
    border-radius: 8px;
    cursor: pointer;
    font-size: 0;

    &:hover {
      background: @black007;
    }
  }

  &__scroll-area {
    position: relative;
    margin: auto;
    width: 100%;
    max-height: 455px;
    margin: 0 0 8px 0;
    padding: 0 !important;
    box-sizing: border-box;

    &.ps--active-y {
      padding-right: 0;
    }

    .ps__rail-y {
      right: 4px !important;
    }
  }
}
</style>
