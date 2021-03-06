<template>
	<div>
		<h1>{{ slice.meta.title }}</h1>
		<p>{{ slice.meta.description }}</p>
		<!-- <nuxt-link :to="`/examples/nuxt/${slice.displayName}`">
			<img class="sample-image" :src="`/components/${slice.displayName}.png`" />
		</nuxt-link>-->
		<div
			class="card"
			@mouseover="mouseover"
			@mouseleave="mouseleave"
			@click="$router.push(`/examples/nuxt/${slice.displayName}`)"
		>
			<!-- <div
				class="image-block"
				:style="{
					backgroundImage: `url('/components/${slice.displayName}.png')`
				}"
			/>-->
			<img class="image-block" :src="`/components/${slice.displayName}.png`" />
			<div :class="
					`hover-block ${hover || isInMyList ? 'hover-block--hovered' : ''}`
				">
				<h5>View Demo</h5>
			</div>
		</div>
		<prismic-rich-text :field="document.install_info_title" />
		<prismic-rich-text :field="document.install_info_text" />
		<!-- <div class="clipboard" @click.stop.prevent="copyCommand">
			<p class="embed-text">import {{ slice.displayName }}</p>
			<img src="../../static/clipboard.svg" />
			<input id="command-to-copy" type="hidden" :value="`import ${slice.displayName}`" />
		</div> -->
		<prismic-rich-text :field="document.info_tagline" />
		<MarkDownBox
			id="markdown-box"
			:edit-url="createEditUrl()"
			style="min-height: 80vh; margin-top: 4em; word-spacing: 0px;"
		>{{ slice.readme }}</MarkDownBox>
		<div v-if="hasSandbox" style="width: 100%; margin-top: 3em;" v-html="sandbox" />
	</div>
</template>

<script>
import Prismic from 'prismic-javascript'
import PrismicConfig from '~/prismic.config.js'

import * as Slices from '@/../src/slices'
import { createSlice, sliceRoute } from '~/utils'

import MarkDownBox from '@/components/MarkDownBox'

const lst = Object.keys(Slices)
	.filter(e => e !== 'SliceZone')
	.map(createSlice)
	.filter(e => e) // eslint-disable-line

export default {
	layout: 'complib',
	components: {
		...Slices,
		MarkDownBox
	},
	async fetch({ store }) {
		await store.dispatch('slices/init')
	},
	async asyncData({ params, error, req }) {
		try {
			// Fetching the API object
			const api = await Prismic.getApi(PrismicConfig.apiEndpoint, { req })

			// Query to get the main menu content
			let document = {}
			const page = await api.getSingle('sample_pages')
			document = page.data

			return {
				// Page content
				document
			}
		} catch (e) {
			error({ statusCode: 404, message: 'Page not found' })
		}
	},
	data() {
		return {
			document: this.document,
			lst,
			slice: createSlice(this.$route.params.slug),
			exampleFolder: `${sliceRoute(this.$route.params.slug)}/examples/nuxt/`,
			hover: false
		}
	},
	computed: {
		hasSandbox: ({ slice, $router }) =>
			$router.resolve({ name: `examples-${slice.displayName}` }).href !== '/',
		sandbox: ({ slice: { displayName } }) =>
			`<iframe src="https://codesandbox.io/embed/github/hypervillain/community/tree/master/?autoresize=1&fontsize=14&initialpath=%2Fexamples%2F${displayName}&module=%2Fwebsite%2Fpages%2Fexamples%2F${displayName}.vue&moduleview=0" title="community" allow="geolocation; microphone; camera; midi; vr; accelerometer; gyroscope; payment; ambient-light-sensor; encrypted-media" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>`
	},
	methods: {
		mouseover() {
			this.hover = true
		},
		mouseleave() {
			this.hover = false
		},
		createEditUrl() {
			const base =
				'https://github.com/prismicio/slice-machine/tree/master/src/slices/'
			return `${base}${this.slice.displayName}/README.md`
		},
		copyCommand() {
			const commandToCopy = document.querySelector('#command-to-copy')
			commandToCopy.setAttribute('type', 'text')
			commandToCopy.select()

			try {
				const successful = document.execCommand('copy')
				const msg = successful ? 'successful' : 'unsuccessful'
				alert('Command was copied ' + msg)
			} catch (err) {
				alert('Oops, unable to copy')
			}

			/* unselect the range */
			commandToCopy.setAttribute('type', 'hidden')
			window.getSelection().removeAllRanges()
		}
	},
	validate: ({ params: { slug } }) =>
		process.env.NODE_ENV === 'production' ? createSlice(slug) !== null : true
}
</script>

<style lang="scss" scoped>
@import '../../style/variables.scss';

.sample-image {
	max-width: 100%;
	border: 1px solid $grey-secondary;
	border-radius: 5px;
	margin: 32px 0;
}
.card {
	box-sizing: border-box;
	background: #fff;
	border: 1px solid $grey-secondary;
	border-radius: 3px;
	position: relative;
	width: 100%;
	margin: 32px 0;
	cursor: pointer;

	.image-block {
		width: 100%;
		background-repeat: no-repeat;
		background-position: center;
		background-size: cover;
		border-radius: 3px;
	}

	.hover-block {
		display: none;
		position: absolute;
		top: 0;
		bottom: 0;
		left: 0;
		right: 0;
		height: 100%;
		width: 100%;
		border-radius: 3px;
		background-image: linear-gradient(
			0deg,
			rgba(104, 104, 104, 0.95),
			rgba(104, 104, 104, 0.95)
		);
		h5 {
			text-align: center;
			color: #fff;
			text-shadow: 1px 1px #000;
			font-weight: bold;
		}
		&--hovered {
			display: flex;
			align-items: center;
			justify-content: center;
			position: absolute;
			flex-direction: column;
		}
	}
}
.clipboard {
	display: flex;
	justify-content: space-between;
	background-color: $black-primary;
	border-radius: 5px;
	padding: 20px;
	margin: 20px 0;
	cursor: pointer;
	img {
		display: none;
		@include rwd(400) {
			margin-left: 30px;
			display: inline-block;
		}
	}
	.embed-text {
		color: #ffffff;
		text-align: center;
		font-size: 13px;
		@include sm {
			font-size: 16px;
		}
	}
	&:hover {
		background-color: $black-secondary;
	}
}
</style>
