<template>
	<div>
		<canvas
            class="waveform"
			ref="canvas"
			:width="width"
			:height="height"
			@click="onClick"
			@mousedown="onMouseDown"
			@mousemove="onMouseMove"
			@mouseleave="onMouseLeave"
		>
		</canvas>
	</div>
</template>

<script>
import variables from '@/assets/scss/_variables.scss'
import mix from '@/utils/mix'
import { BAR_WIDTH, BAR_GAP, TRACK_BARS } from '@/constants'

export default {
	name: 'Waveform',
	props: [
		'progress',
        'data'
	],
	data() {
		const width = TRACK_BARS * (BAR_WIDTH + BAR_GAP)

		return {
			context: null,
			width: width,
			height: 60,
			bars: TRACK_BARS,
			seek: null
		}
	},
	watch: {
		progress: function() {
			this.updateCanvas()
		},
		seek: function() {
			this.updateCanvas()
		}
	},
	methods: {
		updateCanvas() {
			this.context.clearRect(0, -(this.height * (2 / 3)), this.width, this.height)
			this.data.map((length, index) => this.drawBar(index, length))
		},
		drawBar(index, length) {
			const height = length * ((this.height - 2) * (2 / 3))

			this.drawRoundedLine(index, height)
			this.drawRoundedLine(index, height, true)

		},
		calculateColor(index, reverse) {
			if(this.progress < index / this.bars) {
				if(this != null && this.seek > index / this.bars) {
					if(!reverse) {
						return variables.waveform_primary_seek
					} else {
						return variables.waveform_secondary_seek
					}
				}

				if(!reverse) {
					return variables.waveform_primary
				} else {
					return variables.waveform_secondary
				}
			} else if(this.progress > (index + 1) / this.bars) {
				if(this.seek != null && this.seek < index / this.bars) {
					if(!reverse) {
						return variables.waveform_primary_seek
					} else {
						return variables.waveform_secondary_seek
					}
				}

				if(!reverse) {
					return variables.waveform_primary_active
				} else {
					return variables.waveform_secondary_active
				}
			} else {
				if(this.seek != null) {
					if(!reverse) {
						return variables.waveform_primary_seek
					} else {
						return variables.waveform_secondary_seek
					}
				}

				const ratio = (this.progress - (index / this.bars)) * this.bars

				if(!reverse) {
					return mix(variables.waveform_primary_active, variables.waveform_primary, ratio)
				} else {
					return mix(variables.waveform_secondary_active, variables.waveform_secondary, ratio)
				}
			}
		},
		drawRoundedLine(index, length, reverse = false) {
			const x = (index * (BAR_WIDTH + BAR_GAP)) + (BAR_WIDTH / 2)
			const y = 0
			const magnitude = reverse ? (length - 1) / 2 : -length 
			const color = this.calculateColor(index, reverse)

			this.context.strokeStyle = color
			this.context.fillStyle = color
			this.context.lineWidth = 1
			this.context.lineCap = 'butt'

			this.context.beginPath()
			this.context.arc(x, magnitude, 1, 0, Math.PI, !reverse)
			this.context.fill()
			this.context.stroke()

			this.context.lineWidth = BAR_WIDTH

			this.context.beginPath()
			this.context.moveTo(x, reverse ? y + 1 : y)
			this.context.lineTo(x, magnitude)
			this.context.stroke()
		},
		onClick(e) {
			const rect = e.target.getBoundingClientRect()

			const x = e.clientX - rect.left
			const progress = x / this.width

			this.$emit('set-progress', progress)
		},
		onMouseDown(e) {
			const rect = e.target.getBoundingClientRect()

            this.$emit('pause-track')

			document.onmouseup = () => {
				document.onmouseup = null
				document.onmousemove = null

                this.$emit('play-track')
			}

			document.onmousemove = (e1) => {
				const x = e1.clientX - rect.left
				const progress = x / this.width

				this.$emit('set-progress', progress)
			}
		},
		onMouseMove(e) {
			const rect = e.target.getBoundingClientRect()

			const x = e.clientX - rect.left

			this.seek = x / this.width
		},
		onMouseLeave() {
			this.seek = null
		}
	},
	mounted() {
		const canvas = this.$refs['canvas']
		this.context = canvas.getContext('2d')
		this.context.translate(0, this.height * (2 / 3))

		this.updateCanvas()
	}
}
</script>

<style lang="scss" scoped>
canvas.waveform {
    cursor: pointer;
}
</style>