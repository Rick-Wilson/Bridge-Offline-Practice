<template>
  <div class="app">
    <header class="app-header">
      <h1>Bridge Bidding Practice</h1>
      <div class="stats" v-if="practice.state.correctCount + practice.state.wrongCount > 0">
        <span class="correct">✓ {{ practice.state.correctCount }}</span>
        <span class="wrong">✗ {{ practice.state.wrongCount }}</span>
      </div>
    </header>

    <main class="app-main">
      <!-- File loader when no deals -->
      <div v-if="!deals.length" class="no-deals">
        <h2>Load Practice Deals</h2>
        <p>Select a PBN file to start practicing:</p>
        <input
          type="file"
          accept=".pbn"
          @change="onFileSelect"
          ref="fileInput"
        />
        <div v-if="bundledFiles.length" class="bundled-files">
          <p>Or choose a bundled set:</p>
          <button
            v-for="file in bundledFiles"
            :key="file.name"
            class="bundled-btn"
            @click="loadBundledFile(file)"
          >
            {{ file.name }}
          </button>
        </div>
      </div>

      <!-- Practice interface -->
      <template v-else>
        <!-- Deal info -->
        <DealInfo
          :boardNumber="currentDeal?.boardNumber"
          :dealer="currentDeal?.dealer"
          :vulnerable="currentDeal?.vulnerable"
          :contract="currentDeal?.contract"
          :declarer="currentDeal?.declarer"
          :showContract="practice.state.auctionComplete"
          :title="dealTitle"
        />

        <!-- Bridge table with hands -->
        <BridgeTable
          :hands="currentDeal?.hands || {}"
          :hiddenSeats="practice.state.auctionComplete ? [] : practice.hiddenSeats.value"
          :showHcp="practice.state.auctionComplete"
        />

        <!-- Auction and bidding area -->
        <div class="bidding-area">
          <AuctionTable
            :bids="practice.state.displayedBids"
            :dealer="currentDeal?.dealer || 'N'"
            :currentBidIndex="practice.state.currentBidIndex"
            :wrongBidIndex="practice.state.wrongBidIndex"
            :correctBidIndex="practice.state.correctBidIndex"
            :showTurnIndicator="practice.currentBidHasPrompt.value"
          />

          <!-- Bidding box (only show when there's a prompt requiring user input) -->
          <div v-if="!practice.state.auctionComplete && !practice.state.wrongBid && practice.currentBidHasPrompt.value" class="bidding-box-container">
            <!-- Prompt text from PBN commentary -->
            <div v-if="practice.currentPrompt.value?.promptText" class="prompt-text">
              {{ practice.currentPrompt.value.promptText }}
            </div>
            <BiddingBox
              :lastBid="practice.lastContractBid.value"
              :canDouble="practice.canDouble.value"
              :canRedouble="practice.canRedouble.value"
              @bid="onBid"
            />
          </div>

          <!-- Feedback panel for wrong bids -->
          <FeedbackPanel
            :visible="!!practice.state.wrongBid"
            type="wrong"
            :wrongBid="practice.state.wrongBid"
            :correctBid="practice.state.correctBid"
            :commentary="practice.currentExplanation.value"
            @continue="onContinue"
          />

          <!-- Auction complete panel -->
          <div v-if="practice.state.auctionComplete" class="auction-complete">
            <h3>Auction Complete</h3>
            <div v-if="currentDeal?.commentary" class="full-commentary">
              {{ currentDeal.commentary }}
            </div>
            <button class="next-deal-btn" @click="nextDeal">
              Next Deal →
            </button>
          </div>
        </div>

        <!-- Navigation -->
        <DealNavigator
          :deals="deals"
          :currentIndex="currentDealIndex"
          @prev="prevDeal"
          @next="nextDeal"
          @goto="gotoDeal"
        />

        <!-- Load different file -->
        <div class="load-another">
          <input
            type="file"
            accept=".pbn"
            @change="onFileSelect"
            id="loadAnother"
            style="display: none"
          />
          <label for="loadAnother" class="load-link">Load different PBN file</label>
        </div>
      </template>
    </main>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import { parsePbn, getDealTitle } from './utils/pbnParser.js'
import { useBiddingPractice } from './composables/useBiddingPractice.js'

import BridgeTable from './components/BridgeTable.vue'
import BiddingBox from './components/BiddingBox.vue'
import AuctionTable from './components/AuctionTable.vue'
import DealInfo from './components/DealInfo.vue'
import DealNavigator from './components/DealNavigator.vue'
import FeedbackPanel from './components/FeedbackPanel.vue'

// Practice state
const practice = useBiddingPractice()

// Deals data
const deals = ref([])
const currentDealIndex = ref(0)

// Bundled files (we'll add these later)
const bundledFiles = ref([
  { name: 'Cue-bid', url: '/data/Cue-bid.pbn' },
  { name: 'Drury', url: '/data/Drury.pbn' }
])

// Current deal
const currentDeal = computed(() => deals.value[currentDealIndex.value] || null)

const dealTitle = computed(() => {
  if (!currentDeal.value) return ''
  return getDealTitle(currentDeal.value)
})

// Load deal when index changes
watch(currentDealIndex, () => {
  if (currentDeal.value) {
    practice.loadDeal(currentDeal.value)
  }
})

// File handling
async function onFileSelect(event) {
  const file = event.target.files[0]
  if (!file) return

  try {
    const content = await file.text()
    const parsed = parsePbn(content)
    if (parsed.length > 0) {
      deals.value = parsed
      currentDealIndex.value = 0
      practice.loadDeal(parsed[0])
      practice.resetStats()
    } else {
      alert('No deals found in the PBN file')
    }
  } catch (err) {
    console.error('Error loading PBN file:', err)
    alert('Error loading PBN file: ' + err.message)
  }
}

async function loadBundledFile(file) {
  try {
    const response = await fetch(file.url)
    if (!response.ok) throw new Error('Failed to fetch file')
    const content = await response.text()
    const parsed = parsePbn(content)
    if (parsed.length > 0) {
      deals.value = parsed
      currentDealIndex.value = 0
      practice.loadDeal(parsed[0])
      practice.resetStats()
    }
  } catch (err) {
    console.error('Error loading bundled file:', err)
    alert('Error loading file: ' + err.message)
  }
}

// Bidding
function onBid(bid) {
  const correct = practice.makeBid(bid)
  // Feedback is handled via reactive state
}

function onContinue() {
  practice.acceptCorrectBid()
}

// Navigation
function prevDeal() {
  if (currentDealIndex.value > 0) {
    currentDealIndex.value--
  }
}

function nextDeal() {
  if (currentDealIndex.value < deals.value.length - 1) {
    currentDealIndex.value++
  }
}

function gotoDeal(index) {
  if (index >= 0 && index < deals.value.length) {
    currentDealIndex.value = index
  }
}
</script>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
  background: #f5f5f5;
  min-height: 100vh;
}

.app {
  max-width: 800px;
  margin: 0 auto;
  padding: 16px;
}

.app-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding-bottom: 12px;
  border-bottom: 2px solid #ddd;
}

.app-header h1 {
  font-size: 24px;
  color: #333;
}

.stats {
  display: flex;
  gap: 16px;
  font-size: 18px;
  font-weight: bold;
}

.stats .correct {
  color: #4caf50;
}

.stats .wrong {
  color: #d32f2f;
}

.app-main {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.no-deals {
  text-align: center;
  padding: 40px;
  background: #fff;
  border-radius: 8px;
}

.no-deals h2 {
  margin-bottom: 16px;
}

.no-deals p {
  margin-bottom: 12px;
  color: #666;
}

.no-deals input[type="file"] {
  margin-bottom: 20px;
}

.bundled-files {
  margin-top: 20px;
  padding-top: 20px;
  border-top: 1px solid #eee;
}

.bundled-btn {
  margin: 4px;
  padding: 10px 20px;
  border: 1px solid #007bff;
  background: #fff;
  color: #007bff;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.bundled-btn:hover {
  background: #007bff;
  color: #fff;
}

.bidding-area {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
}

.bidding-box-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
}

.turn-indicator {
  font-size: 16px;
  font-weight: 500;
  color: #007bff;
}

.prompt-text {
  max-width: 400px;
  padding: 12px 16px;
  background: #e3f2fd;
  border-left: 4px solid #2196f3;
  border-radius: 4px;
  font-size: 14px;
  line-height: 1.5;
  color: #333;
  text-align: left;
  white-space: pre-wrap;
}

.auction-complete {
  text-align: center;
  padding: 20px;
  background: #e8f5e9;
  border-radius: 8px;
  max-width: 500px;
}

.auction-complete h3 {
  color: #4caf50;
  margin-bottom: 12px;
}

.full-commentary {
  text-align: left;
  font-size: 14px;
  line-height: 1.6;
  color: #333;
  background: #fff;
  padding: 16px;
  border-radius: 4px;
  margin-bottom: 16px;
  white-space: pre-wrap;
}

.next-deal-btn {
  padding: 12px 24px;
  border: none;
  background: #4caf50;
  color: white;
  font-size: 16px;
  font-weight: 500;
  border-radius: 4px;
  cursor: pointer;
}

.next-deal-btn:hover {
  background: #388e3c;
}

.load-another {
  text-align: center;
  margin-top: 8px;
}

.load-link {
  color: #666;
  font-size: 13px;
  cursor: pointer;
  text-decoration: underline;
}

.load-link:hover {
  color: #007bff;
}

@media (max-width: 600px) {
  .app-header {
    flex-direction: column;
    gap: 8px;
  }

  .app-header h1 {
    font-size: 20px;
  }
}
</style>
