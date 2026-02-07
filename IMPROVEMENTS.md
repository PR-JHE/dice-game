# 🚀 Guide des Améliorations

Ce document propose des améliorations progressives pour transformer votre MVP en un jeu complet.

## 🎯 Roadmap suggérée

### Phase 1 : Corrections essentielles (1-2 jours)
### Phase 2 : Expérience utilisateur (3-5 jours)
### Phase 3 : Fonctionnalités avancées (1-2 semaines)
### Phase 4 : Social & Communauté (2-3 semaines)

---

## 📍 Phase 1 : Corrections essentielles

### 1.1 Système de salons partagés

**Problème actuel** : Chaque visiteur crée une nouvelle partie

**Solution** : Système de codes de partie

```javascript
// src/App.jsx - Ajoutez ceci

const [roomCode, setRoomCode] = useState(null)

// Générer un code de salle
function generateRoomCode() {
  return Math.random().toString(36).substring(2, 8).toUpperCase()
}

// Modifier handleJoinGame
const handleJoinGame = async (e) => {
  e.preventDefault()
  if (!username.trim()) return

  setIsJoining(true)

  try {
    // Vérifier si un code de salle existe dans l'URL
    const urlParams = new URLSearchParams(window.location.search)
    const urlRoomCode = urlParams.get('room')

    let game
    
    if (urlRoomCode) {
      // Chercher la partie existante
      const { data: existingGames } = await supabase
        .from('games')
        .select('*')
        .eq('room_code', urlRoomCode)
        .eq('status', 'lobby')
        .single()

      if (existingGames) {
        game = existingGames
      } else {
        alert('Partie non trouvée ou déjà commencée')
        return
      }
    } else {
      // Créer une nouvelle partie avec un code
      const code = generateRoomCode()
      const { data } = await supabase
        .from('games')
        .insert({ status: 'lobby', room_code: code })
        .select()
        .single()
      
      game = data
      setRoomCode(code)
      
      // Mettre à jour l'URL sans recharger la page
      window.history.pushState({}, '', `?room=${code}`)
    }

    setGameId(game.id)
    const player = await db.addPlayer(game.id, username.trim())
    setCurrentPlayer(player)
    setScreen('lobby')
  } catch (error) {
    console.error('Error:', error)
  } finally {
    setIsJoining(false)
  }
}
```

**Migration de la base de données** :

```sql
-- Ajouter la colonne room_code
ALTER TABLE games ADD COLUMN room_code TEXT UNIQUE;

-- Créer un index
CREATE INDEX idx_games_room_code ON games(room_code);
```

### 1.2 Bouton "Copier le lien"

```javascript
// src/components/Lobby/Lobby.jsx

function copyInviteLink() {
  const link = window.location.href
  navigator.clipboard.writeText(link)
  alert('Lien copié ! Partagez-le avec vos amis.')
}

// Dans le JSX du Lobby
<div className="bg-blue-50 border border-blue-200 rounded-lg p-4 mb-6">
  <p className="text-sm text-gray-700 mb-2">
    Invitez vos amis en partageant ce lien :
  </p>
  <div className="flex gap-2">
    <input
      type="text"
      value={window.location.href}
      readOnly
      className="flex-1 px-3 py-2 bg-white border rounded-lg text-sm"
    />
    <button
      onClick={copyInviteLink}
      className="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700"
    >
      📋 Copier
    </button>
  </div>
</div>
```

### 1.3 Déconnexion propre

```javascript
// src/App.jsx

const handleLeaveGame = async () => {
  if (currentPlayer?.id) {
    await supabase
      .from('players')
      .update({ is_connected: false })
      .eq('id', currentPlayer.id)
  }
  
  // Reset l'état
  setGameId(null)
  setCurrentPlayer(null)
  setScreen('welcome')
  window.history.pushState({}, '', '/')
}

// Ajouter un bouton "Quitter" dans le Lobby et GameBoard
```

---

## 🎨 Phase 2 : Expérience utilisateur

### 2.1 Sons et effets

```bash
npm install use-sound
```

```javascript
// src/components/shared/Dice.jsx
import useSound from 'use-sound'

export function DiceRoller({ onRoll, disabled }) {
  const [playDice] = useSound('/sounds/dice-roll.mp3')
  const [playDouble] = useSound('/sounds/double.mp3')
  const [playWin] = useSound('/sounds/win.mp3')

  const handleRoll = async () => {
    playDice()
    // ... reste du code
    
    if (isDoubleSix) {
      playWin()
    } else if (isDouble) {
      playDouble()
    }
  }
}
```

### 2.2 Avatars des joueurs

```javascript
// Utiliser DiceBear API pour des avatars générés
function getAvatarUrl(username) {
  return `https://api.dicebear.com/7.x/avataaars/svg?seed=${username}`
}

// Dans le composant PlayerList
<img 
  src={getAvatarUrl(player.username)} 
  alt={player.username}
  className="w-10 h-10 rounded-full"
/>
```

### 2.3 Mode sombre

```javascript
// src/App.jsx
const [darkMode, setDarkMode] = useState(false)

useEffect(() => {
  if (darkMode) {
    document.documentElement.classList.add('dark')
  } else {
    document.documentElement.classList.remove('dark')
  }
}, [darkMode])

// tailwind.config.js
module.exports = {
  darkMode: 'class',
  // ...
}
```

### 2.4 Animations avancées

```javascript
// Animation de particules pour double 6
import Confetti from 'react-confetti'

{isDoubleSix && (
  <Confetti
    width={window.innerWidth}
    height={window.innerHeight}
    recycle={false}
    numberOfPieces={200}
  />
)}
```

### 2.5 Notifications toast

```bash
npm install react-hot-toast
```

```javascript
import toast, { Toaster } from 'react-hot-toast'

// Dans App.jsx
<Toaster position="top-right" />

// Utilisation
toast.success('Équipe créée !')
toast.error('Erreur de connexion')
toast('Nouveau joueur rejoint la partie', { icon: '👋' })
```

---

## 🎮 Phase 3 : Fonctionnalités avancées

### 3.1 Système de points/score

**Nouvelle table** :

```sql
CREATE TABLE team_scores (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  team_id UUID REFERENCES teams(id) ON DELETE CASCADE,
  game_id UUID REFERENCES games(id) ON DELETE CASCADE,
  points INT DEFAULT 0,
  wins INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW()
);
```

**Logique** :

```javascript
// src/lib/gameLogic.js

export function calculatePoints(diceResult) {
  let points = diceResult.sum
  
  if (diceResult.isDoubleSix) points *= 3
  else if (diceResult.isDouble) points *= 2
  
  if (diceResult.isCatin) points = 0
  
  return points
}
```

### 3.2 Chat en temps réel

**Nouvelle table** :

```sql
CREATE TABLE chat_messages (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  game_id UUID REFERENCES games(id) ON DELETE CASCADE,
  player_id UUID REFERENCES players(id) ON DELETE CASCADE,
  message TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);
```

**Composant** :

```javascript
// src/components/Game/ChatBox.jsx
export function ChatBox({ gameId, currentPlayer }) {
  const [messages, setMessages] = useState([])
  const [newMessage, setNewMessage] = useState('')

  useEffect(() => {
    const channel = supabase
      .channel(`chat:${gameId}`)
      .on('postgres_changes', {
        event: 'INSERT',
        schema: 'public',
        table: 'chat_messages',
        filter: `game_id=eq.${gameId}`
      }, (payload) => {
        setMessages(prev => [...prev, payload.new])
      })
      .subscribe()

    return () => supabase.removeChannel(channel)
  }, [gameId])

  const sendMessage = async () => {
    if (!newMessage.trim()) return
    
    await supabase
      .from('chat_messages')
      .insert({
        game_id: gameId,
        player_id: currentPlayer.id,
        message: newMessage.trim()
      })
    
    setNewMessage('')
  }

  return (
    <div className="flex flex-col h-96">
      <div className="flex-1 overflow-y-auto">
        {messages.map(msg => (
          <div key={msg.id} className="mb-2">
            <span className="font-semibold">{msg.username}: </span>
            {msg.message}
          </div>
        ))}
      </div>
      <div className="flex gap-2">
        <input
          value={newMessage}
          onChange={(e) => setNewMessage(e.target.value)}
          onKeyPress={(e) => e.key === 'Enter' && sendMessage()}
          placeholder="Tapez un message..."
          className="flex-1 px-3 py-2 border rounded"
        />
        <button onClick={sendMessage}>Envoyer</button>
      </div>
    </div>
  )
}
```

### 3.3 Historique des parties

```javascript
// src/components/History/GameHistory.jsx
export function GameHistory({ playerId }) {
  const [history, setHistory] = useState([])

  useEffect(() => {
    loadHistory()
  }, [playerId])

  async function loadHistory() {
    const { data } = await supabase
      .from('games')
      .select(`
        *,
        teams!inner(
          *,
          players!inner(id)
        )
      `)
      .eq('teams.players.id', playerId)
      .eq('status', 'finished')
      .order('created_at', { ascending: false })
      .limit(10)
    
    setHistory(data || [])
  }

  return (
    <div>
      <h2>Vos parties récentes</h2>
      {history.map(game => (
        <div key={game.id}>
          {/* Afficher les détails de la partie */}
        </div>
      ))}
    </div>
  )
}
```

### 3.4 Statistiques avancées

```sql
-- Vue pour les statistiques des joueurs
CREATE VIEW player_stats AS
SELECT 
  p.id,
  p.username,
  COUNT(DISTINCT g.id) as games_played,
  COUNT(DISTINCT CASE WHEN t.is_winner THEN g.id END) as games_won,
  SUM(ts.points) as total_points,
  AVG(ts.points) as avg_points
FROM players p
LEFT JOIN teams t ON p.team_id = t.id
LEFT JOIN games g ON p.game_id = g.id
LEFT JOIN team_scores ts ON t.id = ts.team_id
WHERE g.status = 'finished'
GROUP BY p.id, p.username;
```

### 3.5 Reconnexion automatique

```javascript
// src/hooks/useRealtime.js

useEffect(() => {
  const handleReconnect = () => {
    if (currentPlayer?.id) {
      supabase
        .from('players')
        .update({ is_connected: true })
        .eq('id', currentPlayer.id)
    }
  }

  window.addEventListener('online', handleReconnect)
  
  return () => {
    window.removeEventListener('online', handleReconnect)
  }
}, [currentPlayer])
```

---

## 👥 Phase 4 : Social & Communauté

### 4.1 Système d'amis

```sql
CREATE TABLE friendships (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  player1_id UUID REFERENCES players(id),
  player2_id UUID REFERENCES players(id),
  status TEXT DEFAULT 'pending', -- 'pending', 'accepted', 'rejected'
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(player1_id, player2_id)
);
```

### 4.2 Classement global

```javascript
// src/components/Leaderboard/Leaderboard.jsx
export function Leaderboard() {
  const [topPlayers, setTopPlayers] = useState([])

  useEffect(() => {
    loadLeaderboard()
  }, [])

  async function loadLeaderboard() {
    const { data } = await supabase
      .from('player_stats')
      .select('*')
      .order('total_points', { ascending: false })
      .limit(100)
    
    setTopPlayers(data || [])
  }

  return (
    <div>
      <h1>🏆 Classement</h1>
      <table>
        <thead>
          <tr>
            <th>Rang</th>
            <th>Joueur</th>
            <th>Points</th>
            <th>Victoires</th>
          </tr>
        </thead>
        <tbody>
          {topPlayers.map((player, index) => (
            <tr key={player.id}>
              <td>{index + 1}</td>
              <td>{player.username}</td>
              <td>{player.total_points}</td>
              <td>{player.games_won}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  )
}
```

### 4.3 Authentification avec Supabase Auth

```javascript
// Remplacer le simple username par une vraie auth

// src/lib/auth.js
export async function signUp(email, password, username) {
  const { data, error } = await supabase.auth.signUp({
    email,
    password,
    options: {
      data: {
        username: username
      }
    }
  })
  
  return { data, error }
}

export async function signIn(email, password) {
  const { data, error } = await supabase.auth.signInWithPassword({
    email,
    password
  })
  
  return { data, error }
}
```

### 4.4 Mode spectateur

```javascript
// Permettre aux utilisateurs de regarder une partie sans y participer

const [isSpectator, setIsSpectator] = useState(false)

// Dans le lobby
<button onClick={() => setIsSpectator(true)}>
  👁️ Regarder la partie
</button>
```

---

## 🔧 Optimisations techniques

### 1. Compression des événements

Au lieu de stocker chaque événement, stocker uniquement l'état du jeu :

```javascript
// Créer un snapshot de l'état toutes les 10 actions
if (events.length % 10 === 0) {
  await supabase
    .from('game_snapshots')
    .insert({
      game_id: gameId,
      state: JSON.stringify(gameState)
    })
}
```

### 2. Pagination des événements

```javascript
const EVENTS_PER_PAGE = 20

async function loadMoreEvents(offset) {
  const { data } = await supabase
    .from('game_events')
    .select('*')
    .eq('game_id', gameId)
    .order('created_at', { ascending: false })
    .range(offset, offset + EVENTS_PER_PAGE - 1)
  
  return data
}
```

### 3. Lazy loading des composants

```javascript
import { lazy, Suspense } from 'react'

const GameBoard = lazy(() => import('./components/Game/GameBoard'))

<Suspense fallback={<LoadingSpinner />}>
  <GameBoard />
</Suspense>
```

---

## 📱 Version mobile

### React Native Expo

```bash
npx create-expo-app dice-game-mobile
cd dice-game-mobile
npm install @supabase/supabase-js
```

Réutilisez la même logique métier (`gameLogic.js`) !

---

## 🎯 Métriques à suivre

### Supabase Dashboard
- Nombre de parties créées / jour
- Nombre de joueurs actifs
- Durée moyenne d'une partie
- Taux de reconnexion

### Google Analytics (optionnel)

```bash
npm install @vercel/analytics
```

```javascript
import { Analytics } from '@vercel/analytics/react'

function App() {
  return (
    <>
      <YourApp />
      <Analytics />
    </>
  )
}
```

---

## 🚀 Prochaines grandes features

1. **Tournois** : Système de brackets, éliminatoire
2. **Personnalisation** : Thèmes, couleurs d'équipe, skins de dés
3. **Achievements** : Badges, trophées, défis quotidiens
4. **Replay** : Revoir une partie en vidéo
5. **IA** : Jouer contre un bot

---

Choisissez les améliorations qui vous intéressent et implémentez-les progressivement !

Bon développement ! 🚀
