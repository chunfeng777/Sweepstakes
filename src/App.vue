<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref } from 'vue'

type Game = {
  title: string
  players: number
  category: string
  provider: string
  theme: string
  favorite?: boolean
  following?: boolean
  recent?: boolean
  img?: string
}

type ChatEntry =
  | { divider: string }
  | { name: string; time: string; text: string; avatar: string; color: string }

const assets = {
  hero: 'https://www.figma.com/api/mcp/asset/522ea2ae-f00e-44a1-b39e-ef9927b47b7b',
  leBandit: 'https://www.figma.com/api/mcp/asset/b1e936ce-c37c-4a21-9911-e46d40855f6e',
  sugarRush: 'https://www.figma.com/api/mcp/asset/75f6b990-0861-49e1-99cd-479980785a0e',
  toonBiker: 'https://www.figma.com/api/mcp/asset/ac7a83db-482c-418c-861a-09be23900caa',
  logo: 'https://www.figma.com/api/mcp/asset/91b6e09c-b285-4de6-925f-dffa3ec1a9dd',
  avatarA: 'https://www.figma.com/api/mcp/asset/07459f0d-d6aa-4e8e-8abd-5114c05bf086',
  avatarB: 'https://www.figma.com/api/mcp/asset/867b419f-f12e-4ad4-9f8f-40dbc4cb5fa3',
  avatarC: 'https://www.figma.com/api/mcp/asset/6c920563-298a-4b65-a847-5a116fc2ac20',
  user: 'https://www.figma.com/api/mcp/asset/1bf58e01-08c3-45c7-a4c5-de3e8899ee58',
}

const themes: Record<string, { bg: string; accent: string; icon: string }> = {
  candy: { bg: '#44255f', accent: '#ff6fd8', icon: 'BONUS' },
  fish: { bg: '#123a5d', accent: '#38d6ff', icon: 'FISH' },
  myth: { bg: '#2d2d65', accent: '#ffd166', icon: 'ZEUS' },
  book: { bg: '#4a2b18', accent: '#f4b860', icon: 'BOOK' },
  gems: { bg: '#17374f', accent: '#43f5b6', icon: 'GEMS' },
  jackpot: { bg: '#4a1630', accent: '#ffcb45', icon: 'JACKPOT' },
  adventure: { bg: '#193f2b', accent: '#6ee56f', icon: 'QUEST' },
  classic: { bg: '#21385f', accent: '#ff5c7a', icon: '777' },
  live: { bg: '#102f36', accent: '#7dff67', icon: 'LIVE' },
  table: { bg: '#173827', accent: '#f8f9fa', icon: 'TABLE' },
  original: { bg: '#112f46', accent: '#7dff67', icon: 'OKK' },
  wheel: { bg: '#3d2a55', accent: '#a78bfa', icon: 'SPIN' },
}

function chatAvatar(seed: string, bg: string, accent: string) {
  const initials = seed
    .replace(/[^a-zA-Z0-9]/g, '')
    .slice(0, 2)
    .toUpperCase()
  const svg = `
    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 80 80">
      <defs>
        <linearGradient id="a" x1="0" y1="0" x2="1" y2="1">
          <stop offset="0" stop-color="${accent}"/>
          <stop offset=".55" stop-color="${bg}"/>
          <stop offset="1" stop-color="#071722"/>
        </linearGradient>
      </defs>
      <rect width="80" height="80" rx="18" fill="url(#a)"/>
      <circle cx="56" cy="20" r="18" fill="#fff" opacity=".15"/>
      <circle cx="40" cy="34" r="15" fill="#061722" opacity=".72"/>
      <path d="M15 70c3-19 17-28 25-28s22 9 25 28" fill="#061722" opacity=".72"/>
      <text x="40" y="73" text-anchor="middle" font-family="Inter,Arial" font-size="16" font-weight="900" fill="#fff">${initials}</text>
    </svg>`
  return `data:image/svg+xml;charset=UTF-8,${encodeURIComponent(svg)}`
}

function escapeSvg(value: string) {
  return value.replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;')
}

function splitTitle(title: string) {
  const words = title.toUpperCase().replace(/&/g, 'AND').split(' ')
  const lines: string[] = []

  words.forEach((word) => {
    const current = lines[lines.length - 1] ?? ''
    if (!current || `${current} ${word}`.length > 12) lines.push(word)
    else lines[lines.length - 1] = `${current} ${word}`
  })

  return lines.slice(0, 3)
}

function themeArt(themeKey: string, accent: string, title = '') {
  const commonShadow = `filter="url(#softShadow)"`
  const game = title.toLowerCase()

  if (game.includes('sweet bonanza')) {
    return `
      <circle cx="118" cy="83" r="50" fill="#ff8bd8" ${commonShadow}/>
      <circle cx="86" cy="66" r="24" fill="#78e9ff"/>
      <circle cx="151" cy="61" r="22" fill="#fff06a"/>
      <circle cx="120" cy="94" r="38" fill="#ffffff" opacity=".92"/>
      <path d="M82 119c28 28 66 28 94 0" fill="none" stroke="#ffffff" stroke-width="10" stroke-linecap="round"/>
      <path d="M82 119c28 22 66 22 94 0" fill="none" stroke="#7d3cff" stroke-width="5" stroke-linecap="round" opacity=".55"/>`
  }

  if (game.includes('bass') || game.includes('fishin')) {
    return `
      <path d="M58 104c34-45 96-46 129 0-33 45-95 45-129 0Z" fill="#38d6ff" ${commonShadow}/>
      <path d="M59 104 28 77v56Z" fill="#7dff67"/>
      <path d="M105 72c23 20 23 44 0 65" fill="none" stroke="#ffffff" stroke-width="7" opacity=".54"/>
      <circle cx="153" cy="91" r="7" fill="#061722"/>
      <path d="M171 112c-12 13-28 19-48 19" fill="none" stroke="#061722" stroke-width="5" stroke-linecap="round" opacity=".45"/>`
  }

  if (game.includes('gates') || game.includes('olympus') || game.includes('merlin')) {
    return `
      <path d="M125 28 74 114h39l-18 79 78-108h-44l-4-57Z" fill="#ffd166" ${commonShadow}/>
      <path d="M57 160h126M70 141h100M80 121h80" stroke="#ffffff" stroke-width="8" stroke-linecap="round" opacity=".28"/>
      <circle cx="82" cy="78" r="27" fill="#ffffff" opacity=".18"/>
      <circle cx="158" cy="142" r="35" fill="#ffffff" opacity=".12"/>`
  }

  if (game.includes('book')) {
    return `
      <rect x="64" y="42" width="112" height="135" rx="13" fill="#f4b860" ${commonShadow}/>
      <rect x="82" y="61" width="76" height="98" rx="8" fill="#4a2b18" opacity=".78"/>
      <path d="M101 82h39M97 106h47M101 130h38" stroke="#fff4d6" stroke-width="7" stroke-linecap="round"/>
      <circle cx="120" cy="110" r="55" fill="none" stroke="#ffcb45" stroke-width="5" opacity=".18"/>`
  }

  if (game.includes('starburst')) {
    return `
      <path d="M120 34 183 88 156 171H84L57 88Z" fill="#43f5b6" ${commonShadow}/>
      <path d="M57 88h126M84 88l36 83 36-83M96 54l-12 34M144 54l12 34" stroke="#ffffff" stroke-width="5" opacity=".58"/>
      <path d="M120 19v36M120 168v30M42 96h34M164 96h34" stroke="#b9ffef" stroke-width="6" stroke-linecap="round" opacity=".7"/>`
  }

  if (game.includes('mega') || game.includes('coin') || game.includes('quantum')) {
    return `
      <circle cx="120" cy="105" r="68" fill="#ffcb45" ${commonShadow}/>
      <circle cx="120" cy="105" r="48" fill="#4a1630" opacity=".22"/>
      <path d="M81 106h78M120 67v78" stroke="#ffffff" stroke-width="9" stroke-linecap="round" opacity=".75"/>
      <text x="120" y="126" text-anchor="middle" font-family="Inter,Arial" font-size="42" font-weight="900" fill="#ffffff">SC</text>`
  }

  if (game.includes('roulette')) {
    return `
      <circle cx="120" cy="108" r="70" fill="#263443" ${commonShadow}/>
      <circle cx="120" cy="108" r="52" fill="#1f7a49"/>
      <path d="M120 38v140M50 108h140M70 58l100 100M170 58 70 158" stroke="#ffffff" stroke-width="5" opacity=".5"/>
      <circle cx="120" cy="108" r="18" fill="#ff5c7a"/>
      <text x="120" y="116" text-anchor="middle" font-family="Inter,Arial" font-size="18" font-weight="900" fill="#ffffff">0</text>`
  }

  if (game.includes('blackjack') || game.includes('baccarat')) {
    return `
      <ellipse cx="120" cy="122" rx="82" ry="48" fill="#1f7a49" ${commonShadow}/>
      <rect x="70" y="58" width="48" height="72" rx="8" fill="#fff"/>
      <rect x="124" y="58" width="48" height="72" rx="8" fill="#7dff67"/>
      <text x="94" y="104" text-anchor="middle" font-family="Inter,Arial" font-size="34" font-weight="900" fill="#102f36">A</text>
      <text x="148" y="104" text-anchor="middle" font-family="Inter,Arial" font-size="34" font-weight="900" fill="#102f36">K</text>`
  }

  if (game.includes('plinko')) {
    return `
      <path d="M54 62h132l-21 124H75Z" fill="#0b1d2a" opacity=".45" ${commonShadow}/>
      <g fill="#7dff67">${[72,96,120,144,168].map((x) => `<circle cx="${x}" cy="82" r="6"/>`).join('')}${[84,108,132,156].map((x) => `<circle cx="${x}" cy="116" r="6"/>`).join('')}${[72,96,120,144,168].map((x) => `<circle cx="${x}" cy="150" r="6"/>`).join('')}</g>
      <circle cx="120" cy="45" r="17" fill="#ffffff"/>`
  }

  if (game.includes('mines')) {
    return `
      <rect x="57" y="52" width="126" height="126" rx="18" fill="#102f46" ${commonShadow}/>
      <g fill="#253744">${[0,1,2].map((row) => [0,1,2].map((col) => `<rect x="${73 + col * 35}" y="${68 + row * 35}" width="24" height="24" rx="5"/>`).join('')).join('')}</g>
      <circle cx="120" cy="115" r="18" fill="#ff5c7a"/>
      <path d="M120 91v-14M120 153v-14M96 115H82M158 115h-14" stroke="#ffcb45" stroke-width="6" stroke-linecap="round"/>`
  }

  if (game.includes('dice')) {
    return `
      <rect x="70" y="58" width="100" height="100" rx="18" fill="#ffffff" ${commonShadow}/>
      <g fill="#102f46"><circle cx="96" cy="84" r="8"/><circle cx="144" cy="84" r="8"/><circle cx="120" cy="108" r="8"/><circle cx="96" cy="132" r="8"/><circle cx="144" cy="132" r="8"/></g>`
  }

  if (game.includes('limbo')) {
    return `
      <path d="M67 143c38-66 72-91 110-116-18 49-43 86-91 130Z" fill="#7dff67" ${commonShadow}/>
      <circle cx="151" cy="56" r="20" fill="#ffffff" opacity=".9"/>
      <path d="M59 159c28 5 63-1 92-22" fill="none" stroke="#38d6ff" stroke-width="8" stroke-linecap="round"/>`
  }

  if (game.includes('keno')) {
    return `
      <rect x="55" y="54" width="130" height="120" rx="18" fill="#102f46" ${commonShadow}/>
      <g font-family="Inter,Arial" font-size="18" font-weight="900" text-anchor="middle">${[7,12,19,24,33,41].map((n, i) => `<circle cx="${82 + (i % 3) * 38}" cy="${85 + Math.floor(i / 3) * 45}" r="16" fill="${i % 2 ? '#38d6ff' : '#7dff67'}"/><text x="${82 + (i % 3) * 38}" y="${91 + Math.floor(i / 3) * 45}" fill="#061722">${n}</text>`).join('')}</g>`
  }

  if (game.includes('monopoly') || game.includes('dream') || game.includes('crazy')) {
    return `
      <circle cx="120" cy="110" r="72" fill="#a78bfa" ${commonShadow}/>
      <path d="M120 38v144M48 110h144M69 59l102 102M171 59 69 161" stroke="#ffffff" stroke-width="5" opacity=".58"/>
      <circle cx="120" cy="110" r="22" fill="#3d2a55"/>
      <path d="M120 110 165 70" stroke="#ffcb45" stroke-width="8" stroke-linecap="round"/>`
  }

  const art: Record<string, string> = {
    candy: `
      <circle cx="120" cy="92" r="48" fill="#ff9bd9" ${commonShadow}/>
      <circle cx="91" cy="76" r="24" fill="#7ce8ff" opacity=".95"/>
      <circle cx="150" cy="70" r="20" fill="#fff070" opacity=".95"/>
      <path d="M82 121c28 26 64 26 89 0" fill="none" stroke="#fff" stroke-width="12" stroke-linecap="round" opacity=".9"/>`,
    fish: `
      <path d="M69 105c30-45 91-45 122 0-31 43-92 43-122 0Z" fill="#38d6ff" ${commonShadow}/>
      <path d="M67 105 36 76v58Z" fill="#7dff67"/>
      <circle cx="151" cy="94" r="7" fill="#061722"/>
      <path d="M111 73c20 18 20 46 0 64" fill="none" stroke="#fff" stroke-width="6" opacity=".55"/>`,
    myth: `
      <path d="M130 32 67 131h45l-15 74 75-107h-47l5-66Z" fill="#ffd166" ${commonShadow}/>
      <circle cx="89" cy="89" r="28" fill="#fff" opacity=".18"/>
      <circle cx="161" cy="143" r="34" fill="#fff" opacity=".12"/>`,
    book: `
      <rect x="65" y="45" width="110" height="132" rx="12" fill="#f4b860" ${commonShadow}/>
      <rect x="82" y="61" width="76" height="100" rx="7" fill="#4a2b18" opacity=".72"/>
      <path d="M101 81h39M97 105h47M101 129h38" stroke="#fff" stroke-width="7" stroke-linecap="round" opacity=".85"/>`,
    gems: `
      <path d="M120 42 183 89 157 171H83L57 89Z" fill="#43f5b6" ${commonShadow}/>
      <path d="M57 89h126M84 89l36 82 36-82M96 55l-12 34M144 55l12 34" stroke="#fff" stroke-width="5" opacity=".5"/>`,
    jackpot: `
      <circle cx="120" cy="107" r="68" fill="#ffcb45" ${commonShadow}/>
      <circle cx="120" cy="107" r="48" fill="#4a1630" opacity=".24"/>
      <text x="120" y="126" text-anchor="middle" font-family="Inter,Arial" font-size="52" font-weight="900" fill="#fff">SC</text>`,
    adventure: `
      <circle cx="120" cy="98" r="54" fill="#ffb86c" ${commonShadow}/>
      <path d="M63 81c35-35 79-35 114 0l-19 22H82Z" fill="#6ee56f"/>
      <circle cx="100" cy="105" r="7" fill="#061722"/>
      <circle cx="139" cy="105" r="7" fill="#061722"/>
      <path d="M98 132c15 12 30 12 45 0" fill="none" stroke="#061722" stroke-width="7" stroke-linecap="round"/>`,
    classic: `
      <rect x="57" y="58" width="126" height="94" rx="18" fill="#ff5c7a" ${commonShadow}/>
      <text x="120" y="124" text-anchor="middle" font-family="Inter,Arial" font-size="62" font-weight="900" fill="#fff">777</text>`,
    live: `
      <rect x="55" y="58" width="62" height="88" rx="10" fill="#fff" ${commonShadow}/>
      <rect x="123" y="58" width="62" height="88" rx="10" fill="#7dff67" ${commonShadow}/>
      <text x="86" y="114" text-anchor="middle" font-family="Inter,Arial" font-size="34" font-weight="900" fill="#102f36">A</text>
      <text x="154" y="114" text-anchor="middle" font-family="Inter,Arial" font-size="34" font-weight="900" fill="#102f36">K</text>`,
    table: `
      <ellipse cx="120" cy="111" rx="78" ry="48" fill="#1f7a49" ${commonShadow}/>
      <path d="M53 111h134" stroke="#fff" stroke-width="5" opacity=".28"/>
      <circle cx="120" cy="111" r="24" fill="${accent}"/>
      <text x="120" y="122" text-anchor="middle" font-family="Inter,Arial" font-size="24" font-weight="900" fill="#061722">21</text>`,
    original: `
      <path d="M78 50h84l42 72-42 72H78l-42-72Z" fill="#7dff67" ${commonShadow}/>
      <text x="120" y="134" text-anchor="middle" font-family="Inter,Arial" font-size="46" font-weight="900" fill="#061722">OKK</text>`,
    wheel: `
      <circle cx="120" cy="110" r="70" fill="#a78bfa" ${commonShadow}/>
      <path d="M120 40v140M50 110h140M70 60l100 100M170 60 70 160" stroke="#fff" stroke-width="5" opacity=".55"/>
      <circle cx="120" cy="110" r="18" fill="#3d2a55"/>`,
  }

  return art[themeKey] ?? art.original
}

function coverImage(game: Game) {
  const theme = themes[game.theme] ?? themes.original
  const titleLines = splitTitle(game.title)
  const longestTitleLine = Math.max(...titleLines.map((line) => line.length))
  const titleFontSize = Math.min(titleLines.length > 2 ? 25 : 29, longestTitleLine > 10 ? 25 : 29)
  const titleSvg = titleLines
    .map((line, index) => `<text x="120" y="${224 + index * 30}" text-anchor="middle" font-family="Inter,Arial" font-size="${titleFontSize}" font-weight="900" fill="#ffffff" stroke="#061722" stroke-width="4" paint-order="stroke">${escapeSvg(line)}</text>`)
    .join('')
  const provider = escapeSvg(game.provider.toUpperCase())
  const svg = `
    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 240 320">
      <defs>
        <linearGradient id="bg" x1="0" y1="0" x2="1" y2="1">
          <stop offset="0" stop-color="${theme.accent}"/>
          <stop offset=".48" stop-color="${theme.bg}"/>
          <stop offset="1" stop-color="#061722"/>
        </linearGradient>
        <radialGradient id="glow" cx=".72" cy=".22" r=".62">
          <stop offset="0" stop-color="#ffffff" stop-opacity=".38"/>
          <stop offset="1" stop-color="#ffffff" stop-opacity="0"/>
        </radialGradient>
        <filter id="softShadow" x="-40%" y="-40%" width="180%" height="180%">
          <feDropShadow dx="0" dy="10" stdDeviation="8" flood-color="#061722" flood-opacity=".42"/>
        </filter>
        <linearGradient id="titleFade" x1="0" y1="0" x2="0" y2="1">
          <stop offset="0" stop-color="#061722" stop-opacity="0"/>
          <stop offset="1" stop-color="#061722" stop-opacity=".72"/>
        </linearGradient>
      </defs>
      <rect width="240" height="320" rx="16" fill="url(#bg)"/>
      <circle cx="184" cy="60" r="88" fill="url(#glow)"/>
      <path d="M13 213c44-38 71 32 111-7s63 19 103-22" fill="none" stroke="#fff" stroke-width="10" stroke-linecap="round" opacity=".23"/>
      <path d="M17 218c44-38 70 32 110-7s64 19 104-22" fill="none" stroke="${theme.accent}" stroke-width="7" stroke-linecap="round" opacity=".88"/>
      ${themeArt(game.theme, theme.accent, game.title)}
      <rect x="0" y="176" width="240" height="144" fill="url(#titleFade)"/>
      ${titleSvg}
      <text x="120" y="300" text-anchor="middle" font-family="Inter,Arial" font-size="9" font-weight="800" letter-spacing="1.2" fill="#d2e5f7" opacity=".86">${provider}</text>
    </svg>`
  return `data:image/svg+xml;charset=UTF-8,${encodeURIComponent(svg)}`
}

function promoImage(title: string, tag: string, themeKey: string) {
  const theme = themes[themeKey] ?? themes.original
  const safeTitle = title.replace(/&/g, 'and')
  const svg = `
    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1200 360">
      <defs>
        <linearGradient id="promoBg" x1="0" y1="0" x2="1" y2="1">
          <stop offset="0" stop-color="#061722"/>
          <stop offset=".42" stop-color="${theme.bg}"/>
          <stop offset="1" stop-color="${theme.accent}"/>
        </linearGradient>
        <radialGradient id="flare" cx=".74" cy=".35" r=".55">
          <stop offset="0" stop-color="${theme.accent}" stop-opacity=".7"/>
          <stop offset="1" stop-color="${theme.accent}" stop-opacity="0"/>
        </radialGradient>
      </defs>
      <rect width="1200" height="360" rx="24" fill="url(#promoBg)"/>
      <circle cx="880" cy="110" r="220" fill="url(#flare)"/>
      <g opacity=".22" stroke="#fff" stroke-width="4" fill="none">
        <path d="M720 38 L810 118 L760 212 L652 198 L612 102 Z"/>
        <path d="M942 48 L1054 92 L1028 230 L900 250 L840 138 Z"/>
        <path d="M548 236 C690 160 760 318 920 242 S1110 214 1164 118"/>
      </g>
      <g transform="translate(780 72)">
        <rect x="0" y="0" width="300" height="190" rx="28" fill="#061722" opacity=".44"/>
        <text x="150" y="86" text-anchor="middle" font-family="Inter,Arial" font-size="54" font-weight="900" fill="${theme.accent}">${theme.icon}</text>
        <text x="150" y="136" text-anchor="middle" font-family="Inter,Arial" font-size="28" font-weight="900" fill="#fff">${tag}</text>
      </g>
      <text x="72" y="96" font-family="Inter,Arial" font-size="16" font-weight="900" letter-spacing="4" fill="${theme.accent}">${tag.toUpperCase()}</text>
      <text x="72" y="182" font-family="Inter,Arial" font-size="58" font-weight="900" fill="#fff">${safeTitle}</text>
      <text x="72" y="236" font-family="Inter,Arial" font-size="24" font-weight="700" fill="#d2e5f7">Fresh sweeps picks, bonus drops, and live tables ready now.</text>
    </svg>`
  return `data:image/svg+xml;charset=UTF-8,${encodeURIComponent(svg)}`
}

const allGames = ref<Game[]>([
  { title: 'Sweet Bonanza', players: 248, category: 'Slots', provider: 'Pragmatic Play', theme: 'candy', favorite: true, following: true },
  { title: 'Big Bass Bonanza', players: 231, category: 'Slots', provider: 'Pragmatic Play', theme: 'fish', favorite: true, following: true },
  { title: 'Gates of Olympus', players: 219, category: 'Slots', provider: 'Pragmatic Play', theme: 'myth', favorite: true },
  { title: 'Sugar Rush 1000', players: 184, category: 'New Releases', provider: 'Pragmatic Play', theme: 'candy', following: true, img: assets.sugarRush },
  { title: 'Big Bass Splash', players: 171, category: 'Megaways', provider: 'Pragmatic Play', theme: 'fish', recent: true },
  { title: 'Book of Dead', players: 166, category: 'Slots', provider: 'Play n GO', theme: 'book', favorite: true },
  { title: 'Starburst', players: 153, category: 'Slots', provider: 'NetEnt', theme: 'gems', following: true },
  { title: 'Mega Moolah', players: 146, category: 'Jackpots', provider: 'Microgaming', theme: 'jackpot' },
  { title: 'Gonzo Quest', players: 139, category: 'Megaways', provider: 'NetEnt', theme: 'adventure' },
  { title: 'Bonanza Megaways', players: 127, category: 'Megaways', provider: 'Big Time Gaming', theme: 'gems' },
  { title: 'Fishin Frenzy', players: 118, category: 'Slots', provider: 'Blueprint Gaming', theme: 'fish' },
  { title: 'Le Bandit', players: 110, category: 'Stake Originals', provider: 'Hacksaw Gaming', theme: 'classic', favorite: true, recent: true, img: assets.leBandit },
  { title: 'Plinko', players: 109, category: 'Stake Originals', provider: 'OKK Originals', theme: 'original', following: true },
  { title: 'Mines', players: 107, category: 'Stake Originals', provider: 'OKK Originals', theme: 'original' },
  { title: 'Dice', players: 103, category: 'Stake Originals', provider: 'OKK Originals', theme: 'original', recent: true },
  { title: 'Limbo', players: 99, category: 'Stake Originals', provider: 'OKK Originals', theme: 'original' },
  { title: 'Keno', players: 93, category: 'Stake Originals', provider: 'OKK Originals', theme: 'original' },
  { title: 'Live Blackjack', players: 88, category: 'Live Dealers', provider: 'Evolution', theme: 'live' },
  { title: 'Lightning Roulette', players: 82, category: 'Live Dealers', provider: 'Evolution', theme: 'live' },
  { title: 'Baccarat Squeeze', players: 74, category: 'Live Dealers', provider: 'Evolution', theme: 'live' },
  { title: 'Monopoly Live', players: 69, category: 'Game Shows', provider: 'Evolution', theme: 'wheel' },
  { title: 'Dream Catcher', players: 64, category: 'Game Shows', provider: 'Evolution', theme: 'wheel' },
  { title: 'Crazy Time', players: 62, category: 'Game Shows', provider: 'Evolution', theme: 'wheel' },
  { title: 'Blackjack Neo', players: 59, category: 'Table Games', provider: 'Pragmatic Play Live', theme: 'table' },
  { title: 'European Roulette', players: 56, category: 'Table Games', provider: 'Pragmatic Play Live', theme: 'table' },
  { title: 'Speed Baccarat', players: 51, category: 'Table Games', provider: 'Pragmatic Play Live', theme: 'table' },
  { title: 'Toon Biker', players: 47, category: 'New Releases', provider: 'Spin Reaper', theme: 'adventure', img: assets.toonBiker },
  { title: 'Chill Bill', players: 44, category: 'New Releases', provider: 'Kangaroo Fleet', theme: 'classic' },
  { title: 'Lucky Route', players: 41, category: 'New Releases', provider: 'Spin Reaper', theme: 'adventure', recent: true },
  { title: 'Coin Bonanza', players: 39, category: 'Jackpots', provider: 'Mining Mayhem', theme: 'jackpot' },
  { title: 'Quantum Riches', players: 34, category: 'Jackpots', provider: 'Spin Reaper', theme: 'jackpot' },
  { title: 'Majestic of Merlin', players: 31, category: 'Megaways', provider: 'Kangaroo Fleet', theme: 'myth' },
  { title: 'Fish', players: 181, category: 'Slots', provider: 'Twist Gaming', theme: 'fish', favorite: true },
  { title: 'Buffalo Hold and Win', players: 177, category: 'Slots', provider: 'Booming Games', theme: 'jackpot' },
  { title: 'Le Catcher', players: 81, category: 'Slots', provider: 'Hacksaw Gaming', theme: 'adventure', following: true },
  { title: 'Diamonds Power XXL', players: 146, category: 'Jackpots', provider: 'Playson', theme: 'gems' },
  { title: 'Le Prechaun', players: 207, category: 'Slots', provider: 'Hacksaw Gaming', theme: 'adventure', favorite: true },
  { title: 'Kill Switch', players: 61, category: 'Slots', provider: 'Terminal Games', theme: 'classic' },
  { title: 'Secret Order', players: 24, category: 'Slots', provider: 'Hacksaw Gaming', theme: 'myth' },
  { title: 'Iron Sight', players: 53, category: 'Slots', provider: 'Hacksaw Gaming', theme: 'classic' },
  { title: 'Wanted Dead or A Wild', players: 48, category: 'Megaways', provider: 'Hacksaw Gaming', theme: 'adventure' },
  { title: 'Mother Clucker', players: 43, category: 'Slots', provider: 'Terminal Games', theme: 'adventure' },
  { title: 'Buckshot Benny', players: 35, category: 'Slots', provider: 'Titan Gaming', theme: 'adventure' },
  { title: 'Canyon Gold Bonanza', players: 45, category: 'Megaways', provider: 'Titan Gaming', theme: 'jackpot' },
  { title: 'Soul Catcher', players: 19, category: 'New Releases', provider: 'Knucklehead Syndicate', theme: 'myth' },
  { title: 'Tome of Life', players: 93, category: 'Stake Originals', provider: 'OKK Originals', theme: 'myth' },
  { title: 'Battle Sharks', players: 27, category: 'Slots', provider: 'Pocket Play', theme: 'fish' },
  { title: 'Supercharged Clovers', players: 57, category: 'Jackpots', provider: 'Playson', theme: 'gems' },
  { title: 'Sugar Gates 1K', players: 35, category: 'Megaways', provider: 'Colorful Play', theme: 'candy' },
  { title: 'Thor Hammered', players: 11, category: 'New Releases', provider: 'Massive Studios', theme: 'myth' },
  { title: 'Le Fisherman', players: 54, category: 'Slots', provider: 'Hacksaw Gaming', theme: 'fish' },
  { title: 'The Hermit', players: 16, category: 'Slots', provider: 'Comeback', theme: 'adventure' },
  { title: 'Candy Dash', players: 57, category: 'Slots', provider: 'Paperclip Gaming', theme: 'candy' },
  { title: 'Dragonspire Frostfall', players: 33, category: 'New Releases', provider: 'Spin Reaper', theme: 'myth' },
  { title: 'Wage Slave', players: 21, category: 'New Releases', provider: 'Pixel Forge', theme: 'classic' },
  { title: 'Wizard 2000', players: 28, category: 'Megaways', provider: 'Mining Mayhem', theme: 'myth' },
  { title: 'Jittery Critters', players: 18, category: 'New Releases', provider: 'Massive Studios', theme: 'adventure' },
  { title: 'Soul Rush', players: 22, category: 'New Releases', provider: 'Knucklehead Syndicate', theme: 'gems' },
  { title: 'Full Chamber', players: 36, category: 'Jackpots', provider: 'Phantom Interactive', theme: 'classic' },
  { title: 'Le Cowboy', players: 26, category: 'Slots', provider: 'Hacksaw Gaming', theme: 'adventure' },
  { title: 'Bounty Stacks', players: 49, category: 'Jackpots', provider: 'Mining Mayhem', theme: 'jackpot' },
  { title: 'Diamond Rush', players: 78, category: 'Jackpots', provider: 'Playson', theme: 'gems' },
  { title: 'Wild Wheel', players: 67, category: 'Game Shows', provider: 'Evolution', theme: 'wheel' },
  { title: 'Boom City', players: 55, category: 'Game Shows', provider: 'Pragmatic Play Live', theme: 'wheel' },
  { title: 'Sweet Bonanza CandyLand', players: 73, category: 'Game Shows', provider: 'Pragmatic Play Live', theme: 'candy' },
  { title: 'Crazy Coin Flip', players: 46, category: 'Game Shows', provider: 'Evolution', theme: 'wheel' },
  { title: 'Cash or Crash', players: 38, category: 'Game Shows', provider: 'Evolution', theme: 'wheel' },
  { title: 'Mega Wheel Live', players: 42, category: 'Live Dealers', provider: 'Pragmatic Play Live', theme: 'live' },
  { title: 'Immersive Roulette', players: 58, category: 'Live Dealers', provider: 'Evolution', theme: 'live' },
  { title: 'Live Dragon Tiger', players: 44, category: 'Live Dealers', provider: 'Evolution', theme: 'live' },
  { title: 'Live Sic Bo', players: 33, category: 'Live Dealers', provider: 'Pragmatic Play Live', theme: 'live' },
  { title: 'Speed Blackjack', players: 72, category: 'Live Dealers', provider: 'Evolution', theme: 'live' },
  { title: 'Salon Prive Baccarat', players: 29, category: 'Live Dealers', provider: 'Evolution', theme: 'live' },
  { title: 'Auto Roulette', players: 37, category: 'Live Dealers', provider: 'Pragmatic Play Live', theme: 'live' },
  { title: 'Infinite Blackjack', players: 64, category: 'Table Games', provider: 'Evolution', theme: 'table' },
  { title: 'American Roulette', players: 45, category: 'Table Games', provider: 'Pragmatic Play Live', theme: 'table' },
  { title: 'No Commission Baccarat', players: 39, category: 'Table Games', provider: 'Pragmatic Play Live', theme: 'table' },
  { title: 'Caribbean Stud', players: 28, category: 'Table Games', provider: 'Play n GO', theme: 'table' },
  { title: 'Three Card Poker', players: 25, category: 'Table Games', provider: 'NetEnt', theme: 'table' },
  { title: 'Casino Holdem', players: 22, category: 'Table Games', provider: 'Microgaming', theme: 'table' },
  { title: 'Roulette Royale', players: 52, category: 'Table Games', provider: 'Evolution', theme: 'table' },
  { title: 'Crash', players: 86, category: 'Stake Originals', provider: 'OKK Originals', theme: 'original', recent: true },
  { title: 'Slide', players: 75, category: 'Stake Originals', provider: 'OKK Originals', theme: 'original' },
  { title: 'Tower', players: 68, category: 'Stake Originals', provider: 'OKK Originals', theme: 'original' },
  { title: 'Dragon Tower', players: 63, category: 'Stake Originals', provider: 'OKK Originals', theme: 'original' },
  { title: 'Hilo', players: 58, category: 'Stake Originals', provider: 'OKK Originals', theme: 'original' },
  { title: 'Roulette Original', players: 52, category: 'Stake Originals', provider: 'OKK Originals', theme: 'original' },
  { title: 'Nitro Nights', players: 41, category: 'New Releases', provider: 'Hacksaw Gaming', theme: 'classic' },
  { title: 'Rug Dealer', players: 30, category: 'New Releases', provider: 'Avatar UX', theme: 'adventure' },
])

const navTop = ['Faved Games', 'Following', 'Continue Playing']
const categories = ['Stake Originals', 'Slots', 'Megaways', 'Jackpots', 'Live Dealers', 'Table Games', 'Game Shows', 'New Releases'] as const
type Category = (typeof categories)[number]
const providers = ['All Providers', 'Pragmatic Play', 'Pragmatic Play Live', 'Evolution', 'NetEnt', 'Play n GO', 'Microgaming', 'Big Time Gaming', 'Blueprint Gaming', 'Hacksaw Gaming', 'OKK Originals', 'Spin Reaper', 'Kangaroo Fleet', 'Mining Mayhem']
type SidebarItem =
  | { label: string; icon: string; nav: string }
  | { label: string; icon: string; category: Category }
  | { label: string; icon: string; modal: string; text: string; expand?: boolean }

const sidebarQuickItems: SidebarItem[] = [
  { label: 'Saved Games', icon: 'saved', nav: 'Saved Games' },
  { label: 'Following', icon: 'heart', nav: 'Following' },
  { label: 'Continue Playing', icon: 'continue', nav: 'Continue Playing' },
  { label: 'Games For You', icon: 'star', nav: 'Games For You' },
  { label: 'My Game Play', icon: 'history', nav: 'My Game Play' },
]

const sidebarGameItems: SidebarItem[] = [
  { label: 'Only on Stake', icon: 'stake', category: 'Stake Originals' },
  { label: 'New Releases', icon: 'spark', category: 'New Releases' },
  { label: 'Slot Games', icon: 'slot', category: 'Slots' },
  { label: 'Stake Originals', icon: 'drop', category: 'Stake Originals' },
  { label: 'Live Dealers', icon: 'live', category: 'Live Dealers' },
  { label: 'Burst Games', icon: 'burst', category: 'Megaways' },
  { label: 'Stake Poker', icon: 'poker', modal: 'Stake Poker', text: 'Poker lobby opened. Tournaments, cash tables, and Sit & Go formats would appear here.' },
  { label: 'Feature Spins', icon: 'feature', category: 'Jackpots' },
  { label: 'Table Games', icon: 'table', category: 'Table Games' },
  { label: 'Scratch Cards', icon: 'scratch', modal: 'Scratch Cards', text: 'Scratch card lobby opened. Instant prize cards and limited-time drops would live here.' },
]

const sidebarUtilityItems: SidebarItem[] = [
  { label: 'Promotions', icon: 'gift', modal: 'Promotions', text: 'Promotions center opened. Bonus drops, reloads, and campaign details would live here.', expand: true },
  { label: 'Challenges', icon: 'challenge', modal: 'Challenges', text: 'Challenges opened. Complete missions to earn leaderboard and streak rewards.' },
  { label: 'Blog', icon: 'blog', modal: 'Blog', text: 'Blog opened. News, platform updates, and game guides would appear here.' },
  { label: 'Affiliate', icon: 'affiliate', modal: 'Affiliate', text: 'Affiliate dashboard opened. Referral links, stats, and commission details would appear here.' },
  { label: 'Sponsorships', icon: 'sponsor', modal: 'Sponsorships', text: 'Sponsorships opened. Brand partnerships and creator opportunities would appear here.', expand: true },
  { label: 'Responsible Gaming', icon: 'shield', modal: 'Responsible Gaming', text: 'Responsible gaming tools opened. Session limits, cool-offs, and self-exclusion are available.' },
  { label: 'Live Support', icon: 'support', modal: 'Live Support', text: 'Live support opened. Choose chat, ticket, or help center articles.' },
]

const sidebarPages: Record<string, { title: string; eyebrow: string; intro: string; cards: Array<{ title: string; text: string; metric: string }>; action: string }> = {
  'Stake Poker': {
    title: 'Stake Poker',
    eyebrow: 'Poker Lobby',
    intro: 'Join fast cash tables, scheduled tournaments, and casual Sit & Go rooms with clean table previews and player counts.',
    cards: [
      { title: 'Cash Tables', text: 'Browse no-limit and pot-limit tables by stake, speed, and seated players.', metric: '24 tables' },
      { title: 'Tournaments', text: 'Daily guaranteed events, turbo formats, and final-table highlights.', metric: '$12K GTD' },
      { title: 'Hand History', text: 'Review recent sessions, marked hands, and table performance snapshots.', metric: 'Live stats' },
    ],
    action: 'Open Poker Lobby',
  },
  'Scratch Cards': {
    title: 'Scratch Cards',
    eyebrow: 'Instant Games',
    intro: 'A quick-play area for card-style reveals, limited drops, and prize tiles inspired by instant win formats.',
    cards: [
      { title: 'Daily Scratch', text: 'Fresh card themes rotate every day with simple reveal interactions.', metric: '12 cards' },
      { title: 'Prize Tiles', text: 'Match symbols, uncover multipliers, and chase bonus tiles.', metric: 'Up to 500x' },
      { title: 'Limited Drops', text: 'Seasonal card packs and timed reward windows for returning players.', metric: '3 active' },
    ],
    action: 'View Scratch Cards',
  },
  Promotions: {
    title: 'Promotions',
    eyebrow: 'Rewards Center',
    intro: 'Track bonus drops, reload campaigns, leaderboard races, and limited-time casino promos in one place.',
    cards: [
      { title: 'Bonus Drops', text: 'Claimable drops tied to featured slots and live tables.', metric: '5 live' },
      { title: 'Reload Offers', text: 'Timed reload windows and social casino reward boosts.', metric: 'Daily' },
      { title: 'Leaderboards', text: 'Compete across slots, originals, and table game events.', metric: '$25K pool' },
    ],
    action: 'Browse Promotions',
  },
  Challenges: {
    title: 'Challenges',
    eyebrow: 'Missions',
    intro: 'Complete category missions, streak tasks, and featured-game goals to unlock progress rewards.',
    cards: [
      { title: 'Spin Streak', text: 'Play selected slots across multiple days to keep the streak alive.', metric: '4 days' },
      { title: 'Originals Run', text: 'Try Plinko, Mines, Dice, and Limbo in one mission path.', metric: '6 tasks' },
      { title: 'Live Table Night', text: 'Join live dealer rounds and complete table milestones.', metric: '2 hours' },
    ],
    action: 'Start Challenges',
  },
  Blog: {
    title: 'Blog',
    eyebrow: 'Stories & Guides',
    intro: 'Read game explainers, release notes, sweepstakes updates, and strategy-friendly category guides.',
    cards: [
      { title: 'New Game Notes', text: 'Short release posts for fresh slots, originals, and live tables.', metric: '18 posts' },
      { title: 'How To Play', text: 'Beginner-friendly guides for Megaways, jackpots, and table games.', metric: 'Guides' },
      { title: 'Community', text: 'Highlights from promos, winners, and upcoming feature previews.', metric: 'Weekly' },
    ],
    action: 'Read Blog',
  },
  Affiliate: {
    title: 'Affiliate',
    eyebrow: 'Partner Dashboard',
    intro: 'Manage referral links, campaign assets, performance snapshots, and partner rewards.',
    cards: [
      { title: 'Referral Links', text: 'Generate links for casino, poker, and promotion landing pages.', metric: 'Ready' },
      { title: 'Performance', text: 'Track clicks, signups, and estimated reward activity.', metric: 'Live' },
      { title: 'Creative Kit', text: 'Download banners, logos, and social campaign snippets.', metric: '32 assets' },
    ],
    action: 'Open Affiliate',
  },
  Sponsorships: {
    title: 'Sponsorships',
    eyebrow: 'Brand Partnerships',
    intro: 'A hub for creator collaborations, tournament sponsorships, and featured community events.',
    cards: [
      { title: 'Creator Program', text: 'Apply with audience details and preferred campaign formats.', metric: 'Open' },
      { title: 'Event Slots', text: 'Reserve sponsorship placements for streams and tournaments.', metric: '8 slots' },
      { title: 'Media Pack', text: 'View brand rules, visual assets, and campaign requirements.', metric: 'Updated' },
    ],
    action: 'View Sponsorships',
  },
  'Responsible Gaming': {
    title: 'Responsible Gaming',
    eyebrow: 'Player Tools',
    intro: 'Set reminders, session limits, cooling-off periods, and account safety preferences.',
    cards: [
      { title: 'Session Reminders', text: 'Choose reminder intervals that fit your play habits.', metric: 'Off' },
      { title: 'Play Limits', text: 'Configure daily, weekly, or monthly boundaries.', metric: 'Custom' },
      { title: 'Cool-Off', text: 'Take a timed break with clear return controls.', metric: 'Available' },
    ],
    action: 'Manage Tools',
  },
  'Live Support': {
    title: 'Live Support',
    eyebrow: 'Help Center',
    intro: 'Find quick answers, open a ticket, or start a live support chat for account and game questions.',
    cards: [
      { title: 'Live Chat', text: 'Connect with support for time-sensitive questions.', metric: '< 2 min' },
      { title: 'Help Articles', text: 'Search guides for wallet, rewards, and gameplay questions.', metric: '120+' },
      { title: 'Tickets', text: 'Create a ticket and track follow-up status from your profile.', metric: 'Tracked' },
    ],
    action: 'Contact Support',
  },
}
const categoryGuides = {
  'Stake Originals': {
    title: 'Stake Originals Style Games',
    intro: 'Originals are fast, lightweight games built around simple rules, instant rounds, and transparent risk choices. They are easy to enter, but still let players tune volatility through bet size, multipliers, mines, pins, or target odds.',
    play: 'Pick a stake amount, choose a risk setting, then play quick rounds such as Plinko, Mines, Dice, Limbo, or Keno. Most rounds finish in seconds, making this category ideal for short sessions and testing strategies.',
    mechanics: ['Instant rounds', 'Adjustable risk', 'Simple multiplier targets', 'Low learning curve'],
    examples: ['Plinko', 'Mines', 'Dice', 'Limbo', 'Keno'],
    bestFor: 'Players who want fast decisions, clear odds, and quick replay loops.',
  },
  Slots: {
    title: 'Online Slot Games',
    intro: 'Slots are reel-based games with themes such as candy, fishing, books, gems, and adventures. Wins come from matching symbols, scatters, wilds, free spins, and bonus rounds.',
    play: 'Choose a bet, spin the reels, and watch for matching paylines or cluster wins. Popular titles such as Sweet Bonanza, Big Bass Bonanza, Book of Dead, and Starburst focus on bonus features and volatility.',
    mechanics: ['Wild symbols', 'Scatter bonuses', 'Free spins', 'Tumbling reels'],
    examples: ['Sweet Bonanza', 'Big Bass Bonanza', 'Book of Dead', 'Starburst', 'Fishin Frenzy'],
    bestFor: 'Players who enjoy colorful themes, surprise bonuses, and relaxed spin-by-spin gameplay.',
  },
  Megaways: {
    title: 'Megaways & Expanding Reels',
    intro: 'Megaways games change the number of symbols on each reel every spin, creating many possible ways to win. The board feels more dynamic than a fixed-payline slot.',
    play: 'Spin as usual, but watch the reel height and win ways change each round. Games like Big Bass Splash, Gonzo Quest, and Bonanza Megaways reward chained hits and bonus entries.',
    mechanics: ['Changing reel heights', 'Many win ways', 'Cascading wins', 'High volatility bonuses'],
    examples: ['Big Bass Splash', 'Gonzo Quest', 'Bonanza Megaways', 'Majestic of Merlin'],
    bestFor: 'Players who like slot games with bigger swings and more changing board states.',
  },
  Jackpots: {
    title: 'Jackpot Games',
    intro: 'Jackpot games focus on large prize pools and special bonus triggers. They can feel slower between major hits, but the headline prize potential is the main appeal.',
    play: 'Place eligible bets and look for jackpot symbols, wheel triggers, or progressive prize features. Mega Moolah, Coin Bonanza, and Quantum Riches are positioned around bigger payout moments.',
    mechanics: ['Progressive prizes', 'Jackpot symbols', 'Bonus wheels', 'Prize pool events'],
    examples: ['Mega Moolah', 'Coin Bonanza', 'Quantum Riches'],
    bestFor: 'Players who prefer chasing rare but larger prize outcomes.',
  },
  'Live Dealers': {
    title: 'Live Dealer Games',
    intro: 'Live dealer games stream real table action with human hosts. They combine casino table rules with live pacing, chat energy, and a more social feel.',
    play: 'Join a table, choose a seat or betting area, place wagers during the timer, then watch the dealer resolve the round. Blackjack, Lightning Roulette, and Baccarat Squeeze are common live options.',
    mechanics: ['Live video rounds', 'Betting timers', 'Dealer-hosted tables', 'Shared table results'],
    examples: ['Live Blackjack', 'Lightning Roulette', 'Baccarat Squeeze'],
    bestFor: 'Players who want table-game rules with a real-time casino atmosphere.',
  },
  'Table Games': {
    title: 'Classic Table Games',
    intro: 'Table games are rule-driven experiences such as blackjack, roulette, and baccarat. They rely more on odds, betting decisions, and round structure than slot-style bonuses.',
    play: 'Pick a table, place chips on your chosen outcome, and follow the game rules. Blackjack focuses on card totals, roulette on number/color sections, and baccarat on banker/player outcomes.',
    mechanics: ['Chip betting', 'Defined house rules', 'Card and wheel outcomes', 'Lower visual noise'],
    examples: ['Blackjack Neo', 'European Roulette', 'Speed Baccarat'],
    bestFor: 'Players who like structured rules and strategic bet selection.',
  },
  'Game Shows': {
    title: 'Game Show Style Games',
    intro: 'Game shows turn casino rounds into hosted entertainment with wheels, multipliers, bonus stages, and audience-style excitement.',
    play: 'Choose segments, multipliers, or bonus outcomes before the host starts the round. Monopoly Live, Dream Catcher, and Crazy Time emphasize show pacing and big reveal moments.',
    mechanics: ['Hosted rounds', 'Prize wheels', 'Multiplier segments', 'Bonus stages'],
    examples: ['Monopoly Live', 'Dream Catcher', 'Crazy Time'],
    bestFor: 'Players who want a more social, suspenseful, entertainment-heavy format.',
  },
  'New Releases': {
    title: 'New Release Games',
    intro: 'New releases highlight fresh mechanics, updated art styles, and newly added providers. This category is best for discovering recent lobby additions.',
    play: 'Browse the latest titles, open a game card, and test the feature set with smaller stakes first. New releases may include slots, originals, or experimental bonus formats.',
    mechanics: ['Fresh titles', 'New providers', 'Experimental features', 'Rotating recommendations'],
    examples: ['Sugar Rush 1000', 'Toon Biker', 'Chill Bill', 'Lucky Route'],
    bestFor: 'Players who like trying fresh content before it becomes popular.',
  },
} satisfies Record<Category, { title: string; intro: string; play: string; mechanics: string[]; examples: string[]; bestFor: string }>
const sortOptions = ['Most Popular', 'A-Z', 'Newest', 'Fewest Players']
const betTabs = ['Recent Bets', 'My Bets', 'High Rollers', 'Challenges'] as const
type BetTab = (typeof betTabs)[number]
const carouselSlides = [
  {
    eyebrow: 'EXCLUSIVE OFFER',
    title: 'First Deposit Bonus',
    highlight: '200%',
    text: "Join the world's top community for an unmatched gaming experience.",
    image: assets.hero,
    primary: 'Play Now',
    secondary: 'Learn More',
  },
  {
    eyebrow: 'HOT SLOTS',
    title: 'Sweet Bonanza Drops',
    highlight: 'Daily Wins',
    text: 'Candy reels, tumbling multipliers, and fresh sweeps rewards are featured today.',
    image: promoImage('Sweet Bonanza Drops', 'Hot Slots', 'candy'),
    primary: 'Play Slots',
    secondary: 'View Bonus',
  },
  {
    eyebrow: 'LIVE NOW',
    title: 'Blackjack & Roulette',
    highlight: 'Live Tables',
    text: 'Fast live dealer tables inspired by the most popular sweeps casino lobbies.',
    image: promoImage('Blackjack & Roulette', 'Live Now', 'live'),
    primary: 'Join Table',
    secondary: 'Browse Live',
  },
  {
    eyebrow: 'BIG BASS EVENT',
    title: 'Big Bass Splash',
    highlight: 'Weekend Boost',
    text: 'Fishing slots, Megaways-style picks, and bonus rounds are stacked in the lobby.',
    image: promoImage('Big Bass Splash', 'Event', 'fish'),
    primary: 'Cast Now',
    secondary: 'More Games',
  },
  {
    eyebrow: 'FREE SPIN',
    title: 'Daily Wheel',
    highlight: 'Spin & Claim',
    text: 'Come back every day for reward points, free entries, and rotating featured games.',
    image: promoImage('Daily Wheel', 'Free Spin', 'wheel'),
    primary: 'Spin Now',
    secondary: 'Rewards',
  },
]
const tradePanels = {
  'Recent Bets': {
    summary: 'Live stream of the latest sweeps wagers across OKK STAKES.',
    columns: ['Game', 'User', 'Time', 'Bet Amount', 'Multiplier', 'Payout'],
    rows: [
      ['Blackjack', 'Hidden', '8:15 PM', '13.76', '2.00x', '27.52'],
      ['Dice', 'Hidden', '8:14 PM', '8.40', '1.20x', '10.08'],
      ['Limbo', 'Hidden', '8:13 PM', '21.00', '0.00x', '0.00'],
      ['Sweet Bonanza', 'LuckyMason', '8:12 PM', '18.00', '3.40x', '61.20'],
    ],
  },
  'My Bets': {
    summary: 'Your current session activity and saved outcomes.',
    columns: ['Game', 'Status', 'Time', 'Bet Amount', 'Multiplier', 'Payout'],
    rows: [
      ['Big Bass Splash', 'Settled', '8:11 PM', '12.50', '4.80x', '60.00'],
      ['Plinko', 'Settled', '8:07 PM', '5.00', '1.60x', '8.00'],
      ['Live Blackjack', 'Open', '8:03 PM', '20.00', 'Pending', 'Pending'],
      ['Daily Wheel', 'Reward', '7:59 PM', '0.00', 'Bonus', '+12 pts'],
    ],
  },
  'High Rollers': {
    summary: 'Largest public wins from the last few minutes.',
    columns: ['Game', 'Player', 'Time', 'Bet Amount', 'Multiplier', 'Payout'],
    rows: [
      ['Gates of Olympus', 'ThunderAce', '8:16 PM', '250.00', '18.70x', '4,675.00'],
      ['Mega Moolah', 'RubySpin', '8:13 PM', '140.00', '22.10x', '3,094.00'],
      ['Lightning Roulette', 'TableWolf', '8:10 PM', '320.00', '8.00x', '2,560.00'],
      ['Book of Dead', 'VaultKing', '8:06 PM', '100.00', '16.50x', '1,650.00'],
    ],
  },
  Challenges: {
    summary: 'Daily races and missions players can join right now.',
    columns: ['Challenge', 'Progress', 'Ends', 'Prize Pool', 'Leader', 'Status'],
    rows: [
      ['Slot Sprint', '68%', '02:14:22', '1,500 SC', 'LuckyMason', 'Live'],
      ['Live Table Streak', '42%', '05:08:11', '900 SC', 'CardQueen', 'Live'],
      ['Big Bass Weekend', '76%', '1 day', '3,000 SC', 'BassHunter', 'Live'],
      ['Daily Wheel Run', '25%', '11:42:02', '500 SC', 'NeonNina', 'Live'],
    ],
  },
} satisfies Record<BetTab, { summary: string; columns: string[]; rows: string[][] }>

const chat = ref<ChatEntry[]>([
  { name: 'LuckyMason:', time: '12:02', text: 'Sweet Bonanza is heating up today. Three bonus drops already.', avatar: chatAvatar('LuckyMason', '#123a5d', '#38d6ff'), color: '#38d6ff' },
  { name: 'RubySpin:', time: '12:04', text: 'Big Bass Splash paid me on the last scatter. That was clean.', avatar: chatAvatar('RubySpin', '#4a1630', '#ff6fd8'), color: '#ff6fd8' },
  { name: 'OKKHost:', time: '12:05', text: 'Welcome to OKK STAKES. Keep chat friendly and enjoy the sweeps lobby.', avatar: chatAvatar('OKKHost', '#112f46', '#7dff67'), color: '#7dff67' },
  { divider: 'NEW BONUS DROP' },
  { name: 'TableWolf:', time: '12:07', text: 'Live Blackjack tables feel smooth. Dealer switched fast too.', avatar: chatAvatar('TableWolf', '#173827', '#f8f9fa'), color: '#d3e6f6' },
  { name: 'PixelAce:', time: '12:08', text: 'Anyone tried Gates of Olympus after the new carousel update?', avatar: chatAvatar('PixelAce', '#2d2d65', '#ffd166'), color: '#ffd166' },
  { name: 'BassHunter:', time: '12:09', text: 'I am staying in Megaways. Big Bass and Bonanza are both moving.', avatar: chatAvatar('BassHunter', '#123a5d', '#7dff67'), color: '#7dff67' },
  { name: 'NeonNina:', time: '12:10', text: 'The daily wheel just gave me reward points. Tiny win, still fun.', avatar: chatAvatar('NeonNina', '#3d2a55', '#a78bfa'), color: '#a78bfa' },
  { divider: 'PLAYER CHAT' },
  { name: 'JackpotJay:', time: '12:11', text: 'Mega Moolah is my pick tonight. Good luck everyone.', avatar: chatAvatar('JackpotJay', '#4a1630', '#ffcb45'), color: '#ffcb45' },
  { name: 'CardQueen:', time: '12:12', text: 'Roulette room has a nice pace. I like the live section.', avatar: chatAvatar('CardQueen', '#102f36', '#38d6ff'), color: '#38d6ff' },
])

const activeNav = ref('')
const activeCategory = ref<Category>('Live Dealers')
const activeSidebarKey = ref('Live Dealers')
const activeSidebarPage = ref('')
const activeProvider = ref('All Providers')
const activeSort = ref('Most Popular')
const activeBetTab = ref<BetTab>('Recent Bets')
const query = ref('')
const defaultVisibleCount = 24
const visibleCount = ref(defaultVisibleCount)
const openMenu = ref<'filter' | 'provider' | 'sort' | null>(null)
const selectedGame = ref<Game | null>(null)
const modalTitle = ref('')
const modalText = ref('')
const toast = ref('Ready')
const chatText = ref('')
const chatOpen = ref(true)
const activeSlide = ref(0)
const topMenu = ref<'notifications' | 'settings' | 'profile' | null>(null)
const dragStartX = ref<number | null>(null)
const dragDeltaX = ref(0)
let carouselTimer: number | undefined

const categoryCounts = computed<Record<Category, number>>(() =>
  Object.fromEntries(categories.map((category) => [category, allGames.value.filter((game) => game.category === category).length])) as Record<Category, number>,
)

const filteredGames = computed(() => {
  let games = [...allGames.value]

  if (activeNav.value === 'Faved Games' || activeNav.value === 'Saved Games') games = games.filter((game) => game.favorite)
  if (activeNav.value === 'Following') games = games.filter((game) => game.following)
  if (activeNav.value === 'Continue Playing' || activeNav.value === 'My Game Play') games = games.filter((game) => game.recent)
  if (activeNav.value === 'Games For You') games = games.filter((game) => game.favorite || game.following || game.players >= 120)
  if (!activeNav.value && activeCategory.value) games = games.filter((game) => game.category === activeCategory.value)
  if (activeProvider.value !== 'All Providers') games = games.filter((game) => game.provider === activeProvider.value)
  if (query.value.trim()) {
    const needle = query.value.trim().toLowerCase()
    games = games.filter((game) => game.title.toLowerCase().includes(needle) || game.provider.toLowerCase().includes(needle))
  }

  return games.sort((a, b) => {
    if (activeSort.value === 'A-Z') return a.title.localeCompare(b.title)
    if (activeSort.value === 'Newest') return allGames.value.indexOf(b) - allGames.value.indexOf(a)
    if (activeSort.value === 'Fewest Players') return a.players - b.players
    return b.players - a.players
  })
})

const visibleGames = computed(() => filteredGames.value.slice(0, visibleCount.value))
const canLoadMore = computed(() => visibleCount.value < filteredGames.value.length)
const activeSidebarPageContent = computed(() => activeSidebarPage.value ? sidebarPages[activeSidebarPage.value] : null)
const lobbyTitle = computed(() => activeSidebarPageContent.value?.title || activeNav.value || activeCategory.value || 'All Games')
const currentSlide = computed(() => carouselSlides[activeSlide.value])
const activeTradePanel = computed(() => tradePanels[activeBetTab.value])
const activeCategoryGuide = computed(() => categoryGuides[activeCategory.value] ?? categoryGuides.Slots)

function showToast(message: string) {
  toast.value = message
}

function setCategory(category: Category) {
  activeCategory.value = category
  activeNav.value = ''
  activeSidebarPage.value = ''
  visibleCount.value = defaultVisibleCount
  showToast(`${category} selected`)
}

function setNav(item: string) {
  activeNav.value = item
  activeSidebarPage.value = ''
  visibleCount.value = defaultVisibleCount
  showToast(`${item} selected`)
}

function isSidebarItemActive(item: SidebarItem) {
  return activeSidebarKey.value === item.label
}

function triggerSidebarItem(item: SidebarItem) {
  activeSidebarKey.value = item.label
  if ('category' in item) setCategory(item.category)
  else if ('nav' in item) setNav(item.nav)
  else {
    activeSidebarPage.value = item.modal
    activeNav.value = ''
    query.value = ''
    visibleCount.value = defaultVisibleCount
    showToast(`${item.label} selected`)
  }
}

function setProvider(provider: string) {
  activeProvider.value = provider
  openMenu.value = null
  visibleCount.value = defaultVisibleCount
  showToast(`${provider} applied`)
}

function setSort(sort: string) {
  activeSort.value = sort
  openMenu.value = null
  showToast(`Sorted by ${sort}`)
}

function resetFilters() {
  query.value = ''
  activeProvider.value = 'All Providers'
  activeNav.value = ''
  activeSidebarPage.value = ''
  activeSidebarKey.value = ''
  visibleCount.value = defaultVisibleCount
  openMenu.value = null
  showToast('Filters cleared')
}

function loadMore() {
  visibleCount.value += 8
  showToast('More games loaded')
}

function openGame(game: Game) {
  selectedGame.value = game
  showToast(`${game.title} opened`)
}

function openModal(title: string, text: string) {
  modalTitle.value = title
  modalText.value = text
  topMenu.value = null
}

function closeModal() {
  modalTitle.value = ''
  modalText.value = ''
  selectedGame.value = null
}

function sendChat() {
  const text = chatText.value.trim()
  if (!text) {
    showToast('Type a message first')
    return
  }
  chat.value.push({ name: 'You:', time: 'now', text, avatar: assets.user, color: '#7dff67' })
  chatText.value = ''
  showToast('Message sent')
}

function setSlide(index: number) {
  activeSlide.value = (index + carouselSlides.length) % carouselSlides.length
}

function nextSlide() {
  setSlide(activeSlide.value + 1)
}

function previousSlide() {
  setSlide(activeSlide.value - 1)
}

function toggleTopMenu(menu: 'notifications' | 'settings' | 'profile') {
  topMenu.value = topMenu.value === menu ? null : menu
}

function pauseCarousel() {
  if (carouselTimer) window.clearInterval(carouselTimer)
  carouselTimer = undefined
}

function resumeCarousel() {
  pauseCarousel()
  carouselTimer = window.setInterval(nextSlide, 5000)
}

function startCarouselDrag(event: PointerEvent) {
  if ((event.target as HTMLElement).closest('button')) return
  dragStartX.value = event.clientX
  dragDeltaX.value = 0
  pauseCarousel()
}

function moveCarouselDrag(event: PointerEvent) {
  if (dragStartX.value === null) return
  dragDeltaX.value = event.clientX - dragStartX.value
}

function endCarouselDrag() {
  if (dragStartX.value === null) return
  if (dragDeltaX.value > 60) previousSlide()
  if (dragDeltaX.value < -60) nextSlide()
  dragStartX.value = null
  dragDeltaX.value = 0
  resumeCarousel()
}

function heroPrimaryAction() {
  const slide = currentSlide.value
  if (slide.title.includes('Sweet')) setCategory('Slots')
  else if (slide.title.includes('Blackjack')) setCategory('Live Dealers')
  else if (slide.title.includes('Big Bass')) setCategory('Megaways')
  else if (slide.title.includes('Daily')) showToast('Daily wheel spun: +12 reward points')
  else openModal('Play Now', 'The lobby is ready. Pick a game below to start playing.')
}

onMounted(() => {
  resumeCarousel()
})

onUnmounted(() => {
  if (carouselTimer) window.clearInterval(carouselTimer)
})
</script>

<template>
  <div class="app-shell">
    <header class="topbar">
      <div class="brand-search">
        <button class="brand action-link" type="button" @click="resetFilters">
          <span class="brand-mark" aria-hidden="true"></span>
          <strong>OKK STAKES</strong>
        </button>
        <label class="search">
          <span>Search</span>
          <input v-model="query" aria-label="Search games" placeholder="Search your favorite games..." />
        </label>
      </div>
      <div class="account">
        <div class="balance"><span>$ 0.00</span><button type="button" @click="openModal('Wallet', 'Wallet panel opened. Balance, redeem, and deposit actions would live here.')">+ Wallet</button></div>
        <div class="top-menu">
          <button class="icon-btn bell-icon" type="button" aria-label="Notifications" :aria-expanded="topMenu === 'notifications'" @click="toggleTopMenu('notifications')">
            <span></span>
          </button>
          <div v-if="topMenu === 'notifications'" class="top-popover">
            <strong>Notifications</strong>
            <button type="button" @click="openModal('Bonus Drop', 'Sweet Bonanza and Big Bass Splash have fresh promo boosts today.')">Bonus Drop available</button>
            <button type="button" @click="openModal('VIP Progress', 'You are 64% toward Bronze II. Keep playing to unlock weekly rewards.')">VIP progress updated</button>
            <button type="button" @click="showToast('Notifications marked as read'); topMenu = null">Mark all as read</button>
          </div>
        </div>
        <div class="top-menu">
          <button class="icon-btn gear-icon" type="button" aria-label="Settings" :aria-expanded="topMenu === 'settings'" @click="toggleTopMenu('settings')">
            <span></span>
          </button>
          <div v-if="topMenu === 'settings'" class="top-popover">
            <strong>Quick Settings</strong>
            <button type="button" @click="showToast('Sound effects toggled')">Sound effects</button>
            <button type="button" @click="showToast('Compact cards toggled')">Compact cards</button>
            <button type="button" @click="openModal('Responsible Gaming', 'Session reminders, play limits, and self-exclusion tools are ready here.')">Responsible gaming</button>
          </div>
        </div>
        <div class="top-menu">
          <button class="avatar-btn" type="button" aria-label="Account menu" @click="toggleTopMenu('profile')"><img class="user" :src="assets.user" alt="" /></button>
          <div v-if="topMenu === 'profile'" class="top-popover profile-popover">
            <strong>OKK Player</strong>
            <span>Level 1 - Bronze</span>
            <button type="button" @click="openModal('Account', 'Profile, VIP progress, rewards, and logout controls.')">View profile</button>
            <button type="button" @click="openModal('Rewards', 'Daily wheel, rakeback, and bonus drops are grouped here.')">Rewards</button>
            <button type="button" @click="showToast('Signed out demo action'); topMenu = null">Sign out</button>
          </div>
        </div>
      </div>
    </header>

    <aside class="left-rail stake-rail">
      <div class="rail-switcher">
        <button class="rail-menu" type="button" aria-label="Menu" @click="showToast('Menu opened')"><span></span><span></span><span></span></button>
        <button class="rail-tab active" type="button" @click="resetFilters">Casino</button>
        <button class="rail-tab" type="button" @click="openModal('Poker', 'Poker lobby opened. Tables, tournaments, and hand history would live here.')">Poker</button>
      </div>

      <nav class="stake-nav">
        <section>
          <button v-for="item in sidebarQuickItems" :key="item.label" type="button" :class="{ active: isSidebarItemActive(item) }" @click="triggerSidebarItem(item)">
            <span class="side-icon" :class="`icon-${item.icon}`" aria-hidden="true"></span>{{ item.label }}
          </button>
        </section>

        <section>
          <p>Games</p>
          <button v-for="item in sidebarGameItems" :key="item.label" type="button" :class="{ active: isSidebarItemActive(item) }" @click="triggerSidebarItem(item)">
            <span class="side-icon" :class="`icon-${item.icon}`" aria-hidden="true"></span>{{ item.label }}
          </button>
        </section>

        <section>
          <button v-for="item in sidebarUtilityItems" :key="item.label" type="button" :class="{ active: isSidebarItemActive(item) }" @click="triggerSidebarItem(item)">
            <span class="side-icon" :class="`icon-${item.icon}`" aria-hidden="true"></span>{{ item.label }}<i v-if="'expand' in item && item.expand"></i>
          </button>
        </section>
      </nav>
    </aside>

    <main class="main-content">
      <section
        class="hero-card carousel"
        :class="{ dragging: dragStartX !== null }"
        @mouseenter="pauseCarousel"
        @mouseleave="resumeCarousel(); endCarouselDrag()"
        @pointerdown="startCarouselDrag"
        @pointermove="moveCarouselDrag"
        @pointerup="endCarouselDrag"
        @pointercancel="endCarouselDrag"
      >
        <img :src="currentSlide.image" :alt="currentSlide.title" />
        <div class="hero-shade"></div>
        <div class="hero-copy">
          <span class="eyebrow">{{ currentSlide.eyebrow }}</span>
          <h1>{{ currentSlide.title }} <em>{{ currentSlide.highlight }}</em></h1>
          <p>{{ currentSlide.text }}</p>
          <div class="hero-actions">
            <button class="primary" type="button" @click="heroPrimaryAction">{{ currentSlide.primary }}</button>
            <button class="secondary" type="button" @click="openModal(currentSlide.title, currentSlide.text)">{{ currentSlide.secondary }}</button>
          </div>
        </div>
        <div class="carousel-controls" aria-label="Carousel controls">
          <button type="button" aria-label="Previous slide" @click="previousSlide">Prev</button>
          <button type="button" aria-label="Next slide" @click="nextSlide">Next</button>
        </div>
        <div class="carousel-dots" aria-label="Choose carousel slide">
          <button
            v-for="(_, index) in carouselSlides"
            :key="index"
            type="button"
            :class="{ active: activeSlide === index }"
            :aria-label="`Show slide ${index + 1}`"
            @click="setSlide(index)"
          ></button>
        </div>
      </section>

      <section class="lobby">
        <div class="section-heading">
          <div>
            <span>{{ lobbyTitle }}</span>
            <b v-if="activeSidebarPageContent">{{ activeSidebarPageContent.eyebrow }}</b>
            <b v-else>{{ filteredGames.length }} games</b>
          </div>
          <small>{{ toast }}</small>
        </div>

        <template v-if="activeSidebarPageContent">
          <div class="sidebar-page">
            <div class="sidebar-page-hero">
              <span>{{ activeSidebarPageContent.eyebrow }}</span>
              <h2>{{ activeSidebarPageContent.title }}</h2>
              <p>{{ activeSidebarPageContent.intro }}</p>
              <button type="button" @click="showToast(`${activeSidebarPageContent.title} action selected`)">{{ activeSidebarPageContent.action }}</button>
            </div>
            <div class="sidebar-page-grid">
              <article v-for="card in activeSidebarPageContent.cards" :key="card.title">
                <strong>{{ card.metric }}</strong>
                <h3>{{ card.title }}</h3>
                <p>{{ card.text }}</p>
              </article>
            </div>
          </div>
        </template>

        <template v-else>
          <div class="toolbar">
            <div>
              <div class="menu-wrap">
                <button type="button" @click="openMenu = openMenu === 'filter' ? null : 'filter'">Filter</button>
                <div v-if="openMenu === 'filter'" class="dropdown">
                  <button type="button" @click="resetFilters">Clear all</button>
                  <button v-for="item in navTop" :key="item" type="button" @click="setNav(item)">{{ item }}</button>
                  <button v-for="item in categories" :key="item" type="button" @click="setCategory(item)">{{ item }}</button>
                </div>
              </div>
              <div class="menu-wrap">
                <button type="button" @click="openMenu = openMenu === 'provider' ? null : 'provider'">{{ activeProvider }}</button>
                <div v-if="openMenu === 'provider'" class="dropdown">
                  <button v-for="provider in providers" :key="provider" type="button" :class="{ picked: activeProvider === provider }" @click="setProvider(provider)">{{ provider }}</button>
                </div>
              </div>
            </div>
            <div>
              <button class="grid-icon" type="button" @click="showToast('Grid view selected')">Grid</button>
              <div class="menu-wrap align-right">
                <button type="button" @click="openMenu = openMenu === 'sort' ? null : 'sort'">{{ activeSort }}</button>
                <div v-if="openMenu === 'sort'" class="dropdown">
                  <button v-for="sort in sortOptions" :key="sort" type="button" :class="{ picked: activeSort === sort }" @click="setSort(sort)">{{ sort }}</button>
                </div>
              </div>
            </div>
          </div>
          <div class="game-grid">
            <article v-for="game in visibleGames" :key="game.title" class="game-card" :class="{ selected: selectedGame?.title === game.title }">
              <button class="poster" type="button" @click="openGame(game)">
                <img :src="game.img ?? coverImage(game)" :alt="game.title" />
                <span class="badge">S</span>
              </button>
              <button class="player-count" type="button" @click="openGame(game)"><i></i>{{ game.players }} playing</button>
            </article>
          </div>
          <div class="load-more">
            <span>LOAD MORE</span>
            <button v-if="canLoadMore" type="button" @click="loadMore">View More Games</button>
            <button v-else type="button" @click="showToast('All matching games are visible')">All Games Loaded</button>
          </div>
        </template>
      </section>

      <section class="panel bets">
        <div class="tabs">
          <button v-for="tab in betTabs" :key="tab" type="button" :class="{ active: activeBetTab === tab }" @click="activeBetTab = tab; showToast(`${tab} selected`)">{{ tab }}</button>
        </div>
        <p class="trade-summary">{{ activeTradePanel.summary }}</p>
        <table>
          <thead>
            <tr>
              <th v-for="column in activeTradePanel.columns" :key="column">{{ column }}</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="row in activeTradePanel.rows" :key="row.join('-')">
              <td
                v-for="(cell, index) in row"
                :key="`${row[0]}-${index}-${cell}`"
                :class="{ win: index === 3 || index === 4 || index === 5, muted: cell === 'Pending' }"
              >
                {{ cell }}
              </td>
            </tr>
          </tbody>
        </table>
      </section>

      <section class="info-card">
        <div>
          <span class="info-kicker">{{ activeCategory }}</span>
          <h2>{{ activeCategoryGuide.title }}</h2>
          <p>{{ activeCategoryGuide.intro }}</p>
          <h3>How it plays</h3>
          <p>{{ activeCategoryGuide.play }}</p>
          <p><b>Best for:</b> {{ activeCategoryGuide.bestFor }}</p>
        </div>
        <aside>
          <h3>Core mechanics</h3>
          <ul>
            <li v-for="mechanic in activeCategoryGuide.mechanics" :key="mechanic">{{ mechanic }}</li>
          </ul>
          <h3>Representative games</h3>
          <div class="example-games">
            <button
              v-for="game in activeCategoryGuide.examples"
              :key="game"
              type="button"
              @click="query = game; showToast(`${game} highlighted`)"
            >
              {{ game }}
            </button>
          </div>
          <button type="button" @click="setCategory(activeCategory)">Browse {{ activeCategory }}</button>
        </aside>
      </section>

      <section class="promo-row">
        <div class="discord-card">
          <h2>Join our Official <span>Discord</span></h2>
          <p>Get exclusive codes, participate in daily draws, and exchange tips with millions of players worldwide.</p>
          <button type="button" @click="openModal('Discord', 'Community invite opened. Join draws, tips, and exclusive code drops.')">Join Community</button>
        </div>
        <div class="wheel-card">
          <h3>Daily Wheel</h3>
          <p>Log in every day to get a free spin for a chance to win up to 1,000 USDT rewards.</p>
          <strong>FREE!</strong>
          <button type="button" @click="showToast('Daily wheel spun: +12 reward points')">Spin Now</button>
        </div>
      </section>

      <footer class="footer stake-footer">
        <div class="footer-columns">
          <nav>
            <b>Social Casino</b>
            <button type="button" @click="resetFilters">Games</button>
            <button type="button" @click="setCategory('Slots')">Slots</button>
            <button type="button" @click="openModal('Poker', 'Poker lobby opened. Tables, tournaments, and hand history would live here.')">Poker</button>
            <button type="button" @click="setCategory('Live Dealers')">Live Dealer</button>
            <button type="button" @click="triggerSidebarItem(sidebarGameItems[9])">Scratch Cards</button>
            <button type="button" @click="openModal('Publishers', 'Publisher list opened.')">Publishers</button>
            <button type="button" @click="triggerSidebarItem(sidebarUtilityItems[0])">Promotions</button>
          </nav>
          <nav>
            <b>Stake Originals</b>
            <button type="button" @click="query = 'Plinko'; setCategory('Stake Originals')">Plinko</button>
            <button type="button" @click="query = 'Mines'; setCategory('Stake Originals')">Mines</button>
            <button type="button" @click="query = 'Dice'; setCategory('Stake Originals')">Dice</button>
            <button type="button" @click="query = 'Crash'; setCategory('Stake Originals')">Crash</button>
            <button type="button" @click="query = 'Limbo'; setCategory('Stake Originals')">Limbo</button>
            <button type="button" @click="query = 'Hilo'; setCategory('Stake Originals')">Hilo</button>
            <button type="button" @click="query = ''; setCategory('Stake Originals')">All Stake Originals</button>
          </nav>
          <nav>
            <b>Support</b>
            <button type="button" @click="triggerSidebarItem(sidebarUtilityItems[6])">Help Center</button>
            <button type="button" @click="openModal('Fairness', 'Fairness details opened. Game logic, odds, and transparency notes would live here.')">Fairness</button>
            <button type="button" @click="triggerSidebarItem(sidebarUtilityItems[5])">Gaming Helpline</button>
            <button type="button" @click="triggerSidebarItem(sidebarUtilityItems[6])">Live Support</button>
          </nav>
          <nav>
            <b>About Us</b>
            <button type="button" @click="openModal('VIP Club', 'VIP Club opened. Levels, weekly bonuses, and reward progress would live here.')">VIP Club</button>
            <button type="button" @click="triggerSidebarItem(sidebarUtilityItems[3])">Affiliate</button>
            <button type="button" @click="openModal('Privacy Policy', 'Privacy policy opened.')">Privacy Policy</button>
            <button type="button" @click="openModal('Terms of Service', 'Terms of service opened.')">Terms of Service</button>
            <button type="button" @click="triggerSidebarItem(sidebarUtilityItems[5])">Responsible Play</button>
          </nav>
          <nav>
            <b>FAQ</b>
            <button type="button" @click="openModal('How-to Guides', 'How-to guides opened.')">How-to Guides</button>
            <button type="button" @click="openModal('Social Casino Guide', 'Social casino guide opened.')">Social Casino Guide</button>
            <button type="button" @click="openModal('Slot Game Guide', 'Slot game guide opened.')">Slot Game Guide</button>
            <button type="button" @click="openModal('Live Dealer Guide', 'Live dealer guide opened.')">Live Dealer Guide</button>
            <button type="button" @click="openModal('Customer Support Guide', 'Customer support guide opened.')">Customer Support Guide</button>
            <button type="button" @click="openModal('Stake VIP Guide', 'Stake VIP guide opened.')">Stake VIP Guide</button>
          </nav>
        </div>
        <div class="footer-socials" aria-label="Social links">
          <button type="button" @click="openModal('News', 'News channel opened.')">?</button>
          <button type="button" @click="openModal('X', 'X social channel opened.')">X</button>
          <button type="button" @click="openModal('Instagram', 'Instagram channel opened.')">◎</button>
        </div>
        <div class="footer-rule"></div>
        <div class="footer-legal">
          <p>© 2026 OKK STAKES | 保留所有权利。</p>
          <p>OKK STAKES 由 OKK STAKES LLC所有并运营，注册号为 <strong>2026-002006824</strong>，注册地址为 <strong>Room R, 30 N Gould St, Sheridan, WY 82801, United States</strong>。联系我们：<strong>support@okkstakes.com</strong></p>
          <p>无需购买即可参加抽奖活动。抽奖活动在法律禁止的地区无效。详细规则请见：<button type="button" @click="openModal('Terms of Service', 'Terms of service opened.')">服务条款</button></p>
          <p>OKK Stakes致力于负责任的社会游戏，欲了解更多信息，请访问<button type="button" @click="triggerSidebarItem(sidebarUtilityItems[5])">Gamingaddictsanonymous.org</button></p>
        </div>
        <div class="footer-brand">Stake <span>.us</span></div>
      </footer>
    </main>

    <aside v-if="chatOpen" class="chat-panel">
      <div class="chat-head"><b><i></i>Live Chat</b><span>7,432</span><button type="button" @click="chatOpen = false">Close</button></div>
      <div class="messages">
        <template v-for="entry in chat" :key="'divider' in entry ? entry.divider : entry.name + entry.time">
          <div v-if="'divider' in entry" class="chat-divider">{{ entry.divider }}</div>
          <article v-else class="message">
            <img :src="entry.avatar" alt="" />
            <div>
              <p><b :style="{ color: entry.color }">{{ entry.name }}</b><time>{{ entry.time }}</time></p>
              <span>{{ entry.text }}</span>
            </div>
          </article>
        </template>
      </div>
      <div class="composer">
        <textarea v-model="chatText" placeholder="Type your message..." aria-label="Type your message" @keydown.enter.prevent="sendChat"></textarea>
        <div><span>Emoji Attach</span><button type="button" @click="sendChat">Send</button></div>
      </div>
    </aside>
    <button v-else class="chat-reopen" type="button" @click="chatOpen = true">Open Chat</button>

    <div v-if="selectedGame" class="modal-backdrop" @click.self="closeModal">
      <section class="modal">
        <button class="modal-close" type="button" @click="closeModal">Close</button>
        <h2>{{ selectedGame.title }}</h2>
        <p>{{ selectedGame.provider }} | {{ selectedGame.category }} | {{ selectedGame.players }} players online</p>
        <button class="primary" type="button" @click="showToast(`${selectedGame.title} launch confirmed`)">Launch Game</button>
      </section>
    </div>

    <div v-if="modalTitle" class="modal-backdrop" @click.self="closeModal">
      <section class="modal">
        <button class="modal-close" type="button" @click="closeModal">Close</button>
        <h2>{{ modalTitle }}</h2>
        <p>{{ modalText }}</p>
      </section>
    </div>
  </div>
</template>
