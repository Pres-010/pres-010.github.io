<template>

  <!-- HOME SCREEN -->
  <div v-if="screen === 'home'" class="home">
    <span class="home-badge">Rwanda heritage</span>

    <h1 class="home-title">Find the Hero</h1>
    <p class="home-sub">
      Slide the tiles, assemble the picture,<br>
      and discover a hero of Rwandan history.
    </p>

    <div class="home-preview">
      <div v-for="n in 9" :key="n"
        class="home-preview-tile"
        :class="{ empty: n === 5 }"
      ></div>
    </div>

    <div class="home-steps">
      <div class="home-step">
        <div class="step-icon blue">
          <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
            <rect x="2" y="2" width="7" height="7" rx="2" fill="#185FA5"/>
            <rect x="11" y="2" width="7" height="7" rx="2" fill="#B5D4F4"/>
            <rect x="2" y="11" width="7" height="7" rx="2" fill="#B5D4F4"/>
            <rect x="11" y="11" width="7" height="7" rx="2" fill="#185FA5"/>
          </svg>
        </div>
        <p class="step-num">Step 1</p>
        <p class="step-text">Tap a tile next to the empty space</p>
      </div>
      <div class="home-step">
        <div class="step-icon green">
          <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
            <path d="M4 10h12M12 5l5 5-5 5" stroke="#3B6D11" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </div>
        <p class="step-num">Step 2</p>
        <p class="step-text">Slide tiles to complete the picture</p>
      </div>
      <div class="home-step">
        <div class="step-icon amber">
          <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
            <circle cx="10" cy="10" r="7" stroke="#854F0B" stroke-width="2"/>
            <path d="M7 10l2.5 2.5L14 7" stroke="#854F0B" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </div>
        <p class="step-num">Step 3</p>
        <p class="step-text">Reveal the hero and read their story</p>
      </div>
    </div>

    <div class="home-levels">
      <span class="levels-label">10 levels</span>
      <div v-for="n in 10" :key="n"
        class="level-dot"
        :class="{ active: n === 1 }"
      ></div>
    </div>

    <button class="btn-play" @click="startGame">Play</button>
    <p class="home-footer">Numbers can be toggled on/off during play</p>
  </div>

  <!-- LEVELS SCREEN -->
  <div v-else-if="screen === 'levels'" class="levels-screen">
    <button class="btn-home" @click="screen = 'home'">← Home</button>
    <h2 class="levels-title">Choose a level</h2>
    <p class="levels-sub">Complete each puzzle to unlock the next hero.</p>

    <div class="levels-grid">
      <div
        v-for="(img, index) in images"
        :key="index"
        class="level-card"
        :class="{
          unlocked: index <= unlockedLevel,
          locked: index > unlockedLevel,
          completed: index < unlockedLevel
        }"
        @click="selectLevel(index)"
      >
        <div class="level-thumb">
          <img
            v-if="index <= unlockedLevel"
            :src="img"
            alt="Level thumbnail"
          />
          <div v-else class="level-lock">
            <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
              <rect x="5" y="9" width="10" height="8" rx="2" fill="#94a3b8"/>
              <path d="M7 9V6a3 3 0 0 1 6 0v3" stroke="#94a3b8" stroke-width="2" stroke-linecap="round"/>
            </svg>
          </div>
        </div>

        <div class="level-info">
          <span class="level-number">Level {{ index + 1 }}</span>
          <p v-if="index <= unlockedLevel" class="level-desc">
    {{ descriptions[index] }}
  </p>
          <div class="level-stars">
            <span v-for="n in 3" :key="n" class="level-star">
              {{ levelScores[`level_${index}`] && n <= (levelScores[`level_${index}`].stars || 0) ? '★' : '☆' }}
            </span>
          </div>
          <span v-if="index < unlockedLevel" class="level-status completed">✓ Done</span>
          <span v-else-if="index === unlockedLevel" class="level-status current">Play</span>
          <span v-else class="level-status locked">Locked</span>
        </div>
      </div>
    </div>
  </div>

  <!-- GAME SCREEN -->
  <div v-else>
    <div class="game-nav">
      <button class="btn-home" @click="goHome">← Home</button>
      <button class="btn-home" @click="goToLevels">All levels</button>
    </div>

    <div v-if="!won" class="grid">
      <div
        v-for="(tile, index) in tiles"
        :key="index"
        class="tile"
        :class="{ empty: tile === null }"
        :style="tileStyle(tile)"
        @click="moveTile(index)"
      >
        <span
          v-if="tile !== null && showNumbers"
          class="tile-number"
        >
          {{ tile }}
        </span>
      </div>
    </div>

    <div v-else class="full-image">
      <img :src="images[level]" alt="Completed Puzzle" />
    </div>

    <button class="btn-toggle" @click="showNumbers = !showNumbers">Toggle Numbers</button>
    <p>Moves: {{ moves }} &nbsp;|&nbsp; Time: {{ formattedTime }}</p>
    <button class="btn-restart" @click="shuffle">Restart</button>
    <p>Level: {{ level + 1 }} / {{ images.length }}</p>

    <div v-if="won" class="win">
      <h2>🎉 Puzzle Completed!</h2>

      <p class="win-score">Score: {{ currentScore }}</p>
      <div class="win-stars">
        <span v-for="n in 3" :key="n">
          {{ n <= starsEarned ? '★' : '☆' }}
        </span>
      </div>

      <div class="win-image-wrap">
        <img :src="images[level]" :alt="heroNames[level]" class="win-image" />
        <div class="win-image-label">{{ heroNames[level] }}</div>
      </div>
      <p class="win-description">
  {{ descriptions[level] }}
      </p>
      <p>Moves: {{ moves }}</p>
      <p>{{ facts[level] }}</p>
      <button class="btn-next" @click="nextLevel">Next Level</button>
    </div>
  </div>

</template>

<script>
export default {
  data() {
    return {
      tiles: [],
      screen: 'home',
      unlockedLevel: parseInt(localStorage.getItem('unlockedLevel') || '0'),
      moves: 0,
      showNumbers: true,
      won: false,
      level: 0,

      timer: 0,
      timerInterval: null,
      currentScore: 0,
      levelScores: JSON.parse(localStorage.getItem('levelScores') || '{}'),

      images: [
        '/images/Kigeli.webp',
        '/images/Mutara-III-RUDAHIGWA.jpg',
        '/images/gicanda.jpg',
        '/images/Musinga.png',
        '/images/Queen-Bakayishonga.jpg',
        '/images/ruganzu-king.jpg',
        '/images/agathe-uwilingiyimana.jpeg',
        '/images/Michel.jpg',
        '/images/Kigeli_V_Ndahindurwa.jpg',
        '/images/img10.jpg'
      ],

      heroNames: [
        'King Kigeli',
        'King Mutara III Rudahigwa',
        'Queen Rosalie Gicanda',
        'King Musinga',
        'Queen Bakayishonga',
        'King Ruganzu Ⅱ Ndoli',
        'Prime Minister Agathe Uwilingiyimana',
        'Rwagasana Michel',
        'King Kigeli ⅴ Ndahindurwa',
        'Hero of Rwanda'
      ],

      facts: [
        "King Rwabugiri expanded Rwanda.",
        "Queen Rosalie Gicanda was respected across Rwanda.",
        "Rwanda has a strong oral history tradition.",
        "Traditional leadership shaped unity.",
        "Culture was passed through generations.",
        "Heroes defended the kingdom.",
        "History is preserved through storytelling.",
        "Rwanda values resilience and unity.",
        "Historical leaders shaped modern Rwanda.",
        "Heritage connects all generations."
      ],
      descriptions: [
  "King Kigeli was known for his leadership during challenging times in Rwanda’s history.",
  "Mutara III Rudahigwa played a major role in modernizing Rwanda and promoting unity.",
  "Queen Rosalie Gicanda was admired for her humility and strong cultural influence.",
  "King Musinga ruled during a period of great transition and external influence.",
  "Queen Bakayishonga was known for her intelligence and leadership.",
  "King Ruganzu Ⅱ Ndoli Annexed neighboring kingdoms during expeditions",
  "Agathe Uwilingiyimana was Rwanda’s first female prime minister and a symbol of courage.",
  "Michel contributed to Rwanda’s development in modern times.",
  "This hero represents Rwanda’s resilience and unity.",
  "A symbol of cultural preservation and identity.",
  "A reminder of the strength of Rwanda’s heritage."
]
    }
  },

  computed: {
    starsEarned() {
      if (this.moves <= 60) return 3
      if (this.moves <= 90) return 2
      if (this.moves <= 130) return 1
      return 0
    },

    formattedTime() {
      const m = Math.floor(this.timer / 60)
      const s = this.timer % 60
      return m > 0 ? `${m}m ${s}s` : `${s}s`
    }
  },

  methods: {
    startGame() {
      this.screen = 'levels'
    },

    selectLevel(index) {
      if (index > this.unlockedLevel) return
      this.level = index
      this.screen = 'game'
      this.shuffle()
    },

    tileStyle(tile) {
      if (tile === null) {
        return { background: 'transparent' }
      }
      return {
        backgroundImage: `url('${this.images[this.level]}')`,
        backgroundSize: '300px 300px',
        backgroundPosition: this.getPosition(tile)
      }
    },

    getPosition(tile) {
      const pos = tile - 1
      const x = (pos % 3) * 100
      const y = Math.floor(pos / 3) * 100
      return `-${x}px -${y}px`
    },

    shuffle() {
      let arr = [1, 2, 3, 4, 5, 6, 7, 8, null]
      this.moves = 0
      this.won = false

      for (let i = 0; i < 100; i++) {
        let empty = arr.indexOf(null)
        let neighbors = this.getNeighbors(empty)
        let rand = neighbors[Math.floor(Math.random() * neighbors.length)]
        const temp = arr[empty]
        arr[empty] = arr[rand]
        arr[rand] = temp
      }

      this.tiles = arr

      this.timer = 0
      clearInterval(this.timerInterval)
      this.timerInterval = setInterval(() => { this.timer++ }, 1000)
    },

    moveTile(index) {
      if (this.won) return
      let empty = this.tiles.indexOf(null)
      let neighbors = this.getNeighbors(empty)

      if (neighbors.includes(index)) {
        this.swap(index, empty)
        this.moves++
        this.checkWin()
      }
    },

    swap(i, j) {
      const arr = [...this.tiles]
      const temp = arr[i]
      arr[i] = arr[j]
      arr[j] = temp
      this.tiles = arr
    },

    getNeighbors(i) {
      let n = []
      let r = Math.floor(i / 3)
      let c = i % 3

      if (r > 0) n.push(i - 3)
      if (r < 2) n.push(i + 3)
      if (c > 0) n.push(i - 1)
      if (c < 2) n.push(i + 1)

      return n
    },

    checkWin() {
      const correct = [1, 2, 3, 4, 5, 6, 7, 8, null]
      if (JSON.stringify(this.tiles) === JSON.stringify(correct)) {
        this.won = true
        clearInterval(this.timerInterval)

        const speedBonus = Math.max(0, 200 - this.timer)
        this.currentScore = Math.max(0, 1000 - (this.moves * 10) + speedBonus)

        const key = `level_${this.level}`
        const prev = this.levelScores[key]
        if (!prev || this.currentScore > (prev.score || 0)) {
          this.levelScores[key] = {
            score: this.currentScore,
            stars: this.starsEarned
          }
          localStorage.setItem('levelScores', JSON.stringify(this.levelScores))
        }
      }
    },

    nextLevel() {
      if (this.level >= this.unlockedLevel) {
        this.unlockedLevel = this.level + 1
        localStorage.setItem('unlockedLevel', this.unlockedLevel)
      }

      if (this.level + 1 >= this.images.length) {
        this.level = 0
      } else {
        this.level++
      }

      this.won = false
      this.moves = 0
      this.shuffle()
    },

    goHome() {
      if (this.moves > 0) {
        if (confirm('Go back to home? Your progress will be lost.')) {
          this.screen = 'home'
          this.moves = 0
          this.won = false
          clearInterval(this.timerInterval)
        }
      } else {
        this.screen = 'home'
      }
    },

    goToLevels() {
      if (this.moves > 0) {
        if (confirm('Leave this game? Your progress will be lost.')) {
          this.screen = 'levels'
          this.moves = 0
          this.won = false
          clearInterval(this.timerInterval)
        }
      } else {
        this.screen = 'levels'
      }
    }
  },

  mounted() {
    this.shuffle()
  }
}
</script>

<style>
body {
  font-family: 'Segoe UI', sans-serif;
  background: linear-gradient(to right, #000000, #052e56);
  text-align: center;
  margin: 0;
  padding: 20px;
}

/* ── HOME ── */
.home { max-width: 380px; margin: 0 auto; padding: 2rem 1rem; text-align: center; }
.home-badge { display: inline-block; background: #FAEEDA; color: #854F0B; font-size: 11px; font-weight: 500; padding: 4px 12px; border-radius: 20px; margin-bottom: 1rem; }
.home-title { font-size: 28px; font-weight: 500; color: #1e293b; margin-bottom: 0.4rem; }
.home-sub { font-size: 14px; color: #64748b; line-height: 1.6; margin-bottom: 1.5rem; }
.home-preview { width: 120px; height: 120px; display: grid; grid-template-columns: repeat(3, 1fr); gap: 3px; background: #1e293b; padding: 5px; border-radius: 12px; margin: 0 auto 1.5rem; }
.home-preview-tile { background: #2563eb; border-radius: 5px; }
.home-preview-tile.empty { background: transparent; }
.home-steps { display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; margin-bottom: 1.5rem; }
.home-step { background: #f8fafc; border-radius: 12px; padding: 14px 8px; }
.step-icon { width: 36px; height: 36px; border-radius: 50%; margin: 0 auto 8px; display: flex; align-items: center; justify-content: center; }
.step-icon.blue  { background: #E6F1FB; }
.step-icon.green { background: #EAF3DE; }
.step-icon.amber { background: #FAEEDA; }
.step-num  { font-size: 11px; color: #94a3b8; margin-bottom: 4px; }
.step-text { font-size: 12px; color: #64748b; line-height: 1.4; }
.home-levels { display: flex; align-items: center; justify-content: center; gap: 5px; margin-bottom: 1.5rem; border-top: 1px solid #e2e8f0; padding-top: 1.5rem; }
.levels-label { font-size: 12px; color: #94a3b8; margin-right: 4px; }
.level-dot { width: 8px; height: 8px; border-radius: 50%; background: #C0DD97; }
.level-dot.active { background: #639922; width: 10px; height: 10px; }
.btn-play { width: 100%; padding: 16px; background: #2563eb; color: white; border: none; border-radius: 12px; font-size: 18px; font-weight: 500; cursor: pointer; transition: filter 0.15s, transform 0.15s; }
.btn-play:hover { filter: brightness(1.1); transform: translateY(-2px); }
.home-footer { font-size: 11px; color: #94a3b8; margin-top: 0.75rem; }

/* ── NAV BUTTONS ── */
.btn-home { background: transparent; color: #64748b; border: 1px solid #cbd5e1; padding: 6px 14px; border-radius: 8px; font-size: 13px; font-weight: 400; cursor: pointer; min-height: 36px; transition: background 0.15s, color 0.15s; }
.btn-home:hover { background: #f1f5f9; color: #1e293b; }

/* ── LEVELS SCREEN ── */
.levels-screen { max-width: 380px; margin: 0 auto; padding: 1.5rem 1rem; text-align: center; }
.levels-title { font-size: 22px; font-weight: 500; color: #1e293b; margin: 1rem 0 0.3rem; }
.levels-sub { font-size: 13px; color: #94a3b8; margin-bottom: 1.5rem; }
.levels-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; margin-bottom: 1.5rem; }
.level-card { border-radius: 10px; overflow: hidden; border: 1.5px solid #e2e8f0; cursor: pointer; transition: transform 0.15s, border-color 0.15s; }
.level-card.unlocked:hover { transform: scale(1.04); border-color: #2563eb; }
.level-card.locked { opacity: 0.5; cursor: not-allowed; }
.level-card.completed { border-color: #639922; }
.level-thumb { width: 100%; aspect-ratio: 1; background: #f1f5f9; display: flex; align-items: center; justify-content: center; overflow: hidden; }
.level-thumb img { width: 100%; height: 100%; object-fit: cover; }
.level-lock { display: flex; align-items: center; justify-content: center; width: 100%; height: 100%; }
.level-info { padding: 6px 8px; display: flex; justify-content: space-between; align-items: center; background: white; }
.level-number { font-size: 11px; font-weight: 500; color: #1e293b; }
.level-stars { display: flex; gap: 1px; }
.level-star { font-size: 11px; color: #BA7517; }
.level-status { font-size: 10px; font-weight: 500; padding: 2px 6px; border-radius: 10px; }
.level-status.completed { background: #EAF3DE; color: #3B6D11; }
.level-status.current { background: #E6F1FB; color: #185FA5; }
.level-status.locked { background: #f1f5f9; color: #94a3b8; }
.level-desc {
  font-size: 10px;
  color: #64748b;
  margin-top: 3px;
  line-height: 1.3;

  display: -webkit-box;
  -webkit-line-clamp: 2;   /* show only 2 lines */
  -webkit-box-orient: vertical;
  overflow: hidden;
}

/* ── GAME SCREEN ── */
.game-nav { display: flex; justify-content: space-between; margin-bottom: 1rem; }

h1 { color: #2c3e50; }

.grid {
  width: 300px;
  height: 300px;
  margin: 30px auto;
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 3px;
  background: #000000;
  padding: 6px;
  border-radius: 14px;
}

.full-image img {
  width: 300px;
  height: 300px;
  object-fit: cover;
  border-radius: 14px;
  box-shadow: 0 4px 8px rgba(0,0,0,0.2);
}

.tile {
  height: 100px;
  width: 100%;
  background: #2563eb;
  display: flex;
  justify-content: center;
  align-items: center;
  color: white;
  font-size: 24px;
  cursor: pointer;
  border-radius: 8px;
  transition: transform 0.15s ease, filter 0.15s ease;
  position: relative;
  box-shadow: inset 0 -3px 0 rgba(0,0,0,0.2);
}

.tile:hover { transform: scale(1.08); filter: brightness(1.15); }
.tile:active { transform: scale(0.95); filter: brightness(0.95); box-shadow: inset 0 -1px 0 rgba(0,0,0,0.1); }

.tile-number {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-140%, -140%);
  width: 26px;
  height: 26px;
  border-radius: 50%;
  background: rgb(14, 13, 13);
  opacity: 0.8;
  color: #d1dcee;
  font-size: 13px;
  font-weight: 700;
  pointer-events: none;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* ── BUTTONS ── */
button {
  padding: 10px 18px;
  min-height: 44px;
  margin: 8px;
  border: none;
  border-radius: 8px;
  font-size: 15px;
  font-weight: 500;
  cursor: pointer;
  transition: filter 0.15s, transform 0.15s;
}
button:hover  { filter: brightness(1.1); transform: translateY(-1px); }
button:active { filter: brightness(0.95); transform: scale(0.97); }

.btn-toggle  { background: transparent; color: #2563eb; border: 1.5px solid #2563eb; }
.btn-restart { background: #E24B4A; color: white; }
.btn-next    { background: #1eff00; color: white; }

/* ── WIN SCREEN ── */
.win {
  margin-top: 20px;
  padding: 20px;
  background: #8cf488;
  border: 0.5px solid #C0DD97;
  border-radius: 12px;
  color: #27500A;
  animation: fadeIn 0.5s ease;
}
.win-description {
  font-size: 14px;
  color: #1e293b;
  line-height: 1.6;
  margin-top: 10px;
}
.win-score { font-size: 22px; font-weight: 500; color: #1e293b; margin: 4px 0; }
.win-stars  { font-size: 32px; color: #BA7517; letter-spacing: 6px; margin-bottom: 8px; }

.win-image-wrap {
  position: relative;
  width: 240px;
  margin: 0 auto 1.25rem;
  border-radius: 12px;
  overflow: hidden;
  border: 2px solid #C0DD97;
}

.win-image {
  width: 100%;
  height: 240px;
  object-fit: cover;
  display: block;
  animation: revealImage 0.6s ease;
}

.win-image-label {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  background: rgba(0, 0, 0, 0.55);
  color: white;
  font-size: 13px;
  font-weight: 500;
  padding: 8px 12px;
  text-align: center;
}

/* ── ANIMATIONS ── */
@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to   { opacity: 1; transform: translateY(0); }
}

@keyframes revealImage {
  from { opacity: 0; transform: scale(0.92); }
  to   { opacity: 1; transform: scale(1); }
}
</style>