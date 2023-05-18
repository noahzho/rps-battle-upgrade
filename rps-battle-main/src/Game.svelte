import { Canvas, Layer } from 'svelte-canvas'
import { onMount } from 'svelte/internal'

let width = document.body.clientWidth
let height = document.body.clientHeight

type AgentType = 'rock' | 'paper' | 'scissors'

class Agent {
  x: number
  y: number
  target: Agent
  type: AgentType
  isPaused: boolean

  constructor(x: number, y: number, type: AgentType) {
    this.x = x
    this.y = y
    this.type = type
    this.isPaused = false
  }

  canAttack(type: AgentType) {
    let attackableType: AgentType

    switch (this.type) {
      case 'paper': {
        attackableType = 'rock'
        break
      }
      case 'rock': {
        attackableType = 'scissors'
        break
      }
      case 'scissors': {
        attackableType = 'paper'
        break
      }
    }

    return attackableType == type
  }

  findTarget(agents: Agent[]): Agent {
    let minDistance = Infinity
    let closestAgent: Agent

    for (const agent of agents) {
      if (agent == this || agent.type == this.type || !this.canAttack(agent.type))
        continue

      const distance = Math.sqrt(
        Math.abs(agent.x - this.x) ** 2 + Math.abs(agent.y - this.y) ** 2
      )
      if (distance < minDistance) {
        minDistance = distance
        closestAgent = agent
      }
    }

    return closestAgent
  }

  findAvoid(agents: Agent[]): Agent {
    let minDistance = Infinity
    let closestAgent: Agent

    for (const agent of agents) {
      if (agent == this || agent.type == this.type || this.canAttack(agent.type))
        continue

      const distance = Math.sqrt(
        Math.abs(agent.x - this.x) ** 2 + Math.abs(agent.y - this.y) ** 2
      )
      if (distance < minDistance) {
        minDistance = distance
        closestAgent = agent
      }
    }

    return closestAgent
  }

  updatePosition(agents: Agent[]) {
    if (!this.target || !this.canAttack(this.target.type)) {
      this.target = this.findTarget(agents)
    }

    if (this.isPaused) {
      return
    }

    const speed = this.target ? 2 : -1

    if (!this.target) {
      this.target = this.findAvoid(agents)
    }

    if (!this.target) {
      return
    }

    const dx = this.target.x - this.x
    const dy = this.target.y - this.y
    const distance = Math.sqrt(dx * dx + dy * dy)

    const angle = Math.atan2(dy, dx)
    const vx = speed * Math.cos(angle)
    const vy = speed * Math.sin(angle)

    const newX = this.x + vx
    const newY = this.y + vy

    let collided = false

    for (const agent of agents) {
      if (agent !== this && Math.abs(agent.x - newX) < 30 && Math.abs(agent.y - newY) < 30) {
        if (agent.canAttack(this.type)) {
          collided = true
          if (!this.isPaused) {
            console.log(`Collision detected between ${this.type} and ${agent.type}. Pausing for 1 second.`)
            this.isPaused = true
            setTimeout(() => {
              console.log(`Resuming movement for ${this.type}.`)
              this.type = agent.type
              this.isPaused = false
            }, 1000)
          }
        }
        break
      }
    }

    if (!collided) {
      if (newX < width && newX > 0) {
        this.x = newX
      }

      if (newY < height && newY > 0) {
        this.y = newY
      }
    }
  }
}

export let agentCount = 50

const agents = [
  new Agent(50, 50, 'paper'),
  new Agent(100, 150, 'rock'),
  new Agent(200, 100, 'scissors'),
]

for (let i = 1; i <= agentCount - 3; i++) {
  const random = i % 3

  agents.push(
    new Agent(
      Math.random() * (width - 40 + 20),
      Math.random() * (height - 40 + 20),
      random == 0 ? 'paper' : random == 1 ? 'rock' : 'scissors'
    )
  )
}

onMount(() => {
  const canvas = document.getElementById('canvas')

  const context = canvas.getContext('2d')
  context.font = '24px sans-serif'

  window.addEventListener('resize', () => {
    width = document.body.clientWidth
    height = document.body.clientHeight
  })

  function draw() {
    for (const agent of agents) {
      context.fillText(
        agent.type == 'paper'
          ? '📜'
          : agent.type == 'scissors'
          ? '✂️'
          : agent.type == 'rock'
          ? '🪨'
          : '🪨',
        agent.x,
        agent.y
      )

      agent.updatePosition(agents)
    }
  }

  setInterval(() => {
    context.clearRect(0, 0, width, height)
    draw()
  }, 50)
})
