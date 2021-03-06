<template>
<div class="mk-welcome">
	<button @click="dark">
		<template v-if="$store.state.device.darkmode">%fa:moon%</template>
		<template v-else>%fa:R moon%</template>
	</button>

	<mk-forkit class="forkit"/>

	<div class="body">
		<div class="main block">
			<div>
				<h1 v-if="name != 'Misskey'">{{ name }}</h1>
				<h1 v-else><img :src="$store.state.device.darkmode ? 'assets/title.dark.svg' : 'assets/title.light.svg'" :alt="name"></h1>

				<div class="info">
					<span><b>{{ host }}</b> - <span v-html="'%i18n:@powered-by-misskey%'"></span></span>
					<span class="stats" v-if="stats">
						<span>%fa:user% {{ stats.originalUsersCount | number }}</span>
						<span>%fa:pencil-alt% {{ stats.originalNotesCount | number }}</span>
					</span>
				</div>

				<p class="desc" v-html="description || '%i18n:common.about%'"></p>

				<p class="sign">
					<span class="signup" @click="signup">%i18n:@signup%</span>
					<span class="divider">|</span>
					<span class="signin" @click="signin">%i18n:@signin%</span>
				</p>
			</div>
		</div>

		<div class="announcements block">
			<header>%fa:broadcast-tower% %i18n:@announcements%</header>
			<div>
				<div v-for="announcement in announcements">
					<h1 v-html="announcement.title"></h1>
					<div v-html="announcement.text"></div>
				</div>
			</div>
		</div>

		<div class="photos block">
			<header>%fa:images% %i18n:@photos%</header>
			<div>
				<div v-for="photo in photos" :style="`background-image: url(${photo.thumbnailUrl})`"></div>
			</div>
		</div>

		<div class="nav block">
			<div>
				<mk-nav class="nav"/>
			</div>
		</div>

		<div class="side">
			<div class="trends block">
				<div>
					<mk-trends/>
				</div>
			</div>

			<div class="tl block">
				<header>%fa:comment-alt R% %i18n:@timeline%</header>
				<div>
					<mk-welcome-timeline class="tl" :max="20"/>
				</div>
			</div>
		</div>
	</div>

	<modal name="signup" :class="$store.state.device.darkmode ? 'modal-dark' : 'modal-light'" width="450px" height="auto" scrollable>
		<header class="formHeader">%i18n:@signup%</header>
		<mk-signup class="form"/>
	</modal>

	<modal name="signin" :class="$store.state.device.darkmode ? 'modal-dark' : 'modal-light'" width="450px" height="auto" scrollable>
		<header class="formHeader">%i18n:@signin%</header>
		<mk-signin class="form"/>
	</modal>
</div>
</template>

<script lang="ts">
import Vue from 'vue';
import { host, copyright } from '../../../config';

export default Vue.extend({
	data() {
		return {
			stats: null,
			copyright,
			host,
			name: 'Misskey',
			description: '',
			announcements: [],
			photos: []
		};
	},

	created() {
		(this as any).os.getMeta().then(meta => {
			this.name = meta.name;
			this.description = meta.description;
			this.announcements = meta.broadcasts;
		});

		(this as any).api('stats').then(stats => {
			this.stats = stats;
		});

		const image = [
			'image/jpeg',
			'image/png',
			'image/gif'
		];

		(this as any).api('notes/local-timeline', {
			fileType: image,
			limit: 6
		}).then(notes => {
			const files = [].concat(...notes.map(n => n.files));
			this.photos = files.filter(f => image.includes(f.type)).slice(0, 6);
		});
	},

	methods: {
		signup() {
			this.$modal.show('signup');
		},

		signin() {
			this.$modal.show('signin');
		},

		dark() {
			this.$store.commit('device/set', {
				key: 'darkmode',
				value: !this.$store.state.device.darkmode
			});
		}
	}
});
</script>

<style lang="stylus">
#wait
	right auto
	left 15px

.v--modal-overlay
	background rgba(0, 0, 0, 0.6)

.modal-light
	.v--modal-box
		color #777

		.formHeader
			border-bottom solid 1px #eee

.modal-dark
	.v--modal-box
		background #313543
		color #fff

		.formHeader
			border-bottom solid 1px rgba(#000, 0.2)

.modal-light
.modal-dark
	.form
		padding 24px 48px 48px 48px

	.formHeader
		text-align center
		padding 48px 0 12px 0
		margin 0 48px
		font-size 1.5em

</style>

<style lang="stylus" scoped>
@import '~const.styl'

root(isDark)
	display flex
	min-height 100vh
	//background-color #00070F
	//background-image url('/assets/bg.jpg')
	//background-position center
	//background-size cover

	> .forkit
		position absolute
		top 0
		right 0

	> button
		position fixed
		z-index 1
		bottom 16px
		left 16px
		padding 16px
		font-size 18px
		color isDark ? #fff : #444

	> .body
		display grid
		grid-template-rows 1fr 1fr 64px
		grid-template-columns 1fr 1fr 350px
		gap 16px
		width 100%
		max-width 1200px
		height 100vh
		min-height 1000px
		margin 0 auto
		padding 64px

		.block
			color isDark ? #fff : #444
			background isDark ? #282C37 : #fff
			box-shadow 0 3px 8px rgba(0, 0, 0, 0.2)
			//border-radius 8px
			overflow auto

			> header
				z-index 1
				padding 0 16px
				line-height 48px
				background isDark ? #313543 : #fff

				if !isDark
					box-shadow 0 1px 0px rgba(0, 0, 0, 0.1)

				& + div
					max-height calc(100% - 48px)

			> div
				overflow auto

		> .main
			grid-row 1
			grid-column 1 / 3
			border-top solid 5px $theme-color

			> div
				padding 32px

				> h1
					margin 0

					> img
						margin -8px 0 0 -16px
						max-width 280px

				> .info
					margin 0 auto 16px auto
					width $width
					font-size 14px

					> .stats
						margin-left 16px
						padding-left 16px
						border-left solid 1px isDark ? #fff : #444

						> *
							margin-right 16px

				> .sign
					font-size 120%

					> .divider
						margin 0 16px

					> .signin
					> .signup
						cursor pointer

						&:hover
							color $theme-color

		> .announcements
			grid-row 2
			grid-column 1

			> div
				padding 32px

				> div
					padding 0 0 16px 0
					margin 0 0 16px 0
					border-bottom 1px solid isDark ? rgba(#000, 0.2) : rgba(#000, 0.05)

					> h1
						margin 0
						font-size 1.25em

		> .photos
			grid-row 2
			grid-column 2

			> div
				display grid
				grid-template-rows 1fr 1fr 1fr
				grid-template-columns 1fr 1fr
				gap 8px
				height 100%
				padding 16px

				> div
					//border-radius 4px
					background-position center center
					background-size cover

		> .nav
			display flex
			justify-content center
			align-items center
			grid-row 3
			grid-column 1 / 3
			font-size 14px

		> .side
			display grid
			grid-row 1 / 4
			grid-column 3
			grid-template-rows 1fr 350px
			grid-template-columns 1fr
			gap 16px

			> .tl
				grid-row 1
				grid-column 1
				overflow auto

			> .trends
				grid-row 2
				grid-column 1
				padding 8px

.mk-welcome[data-darkmode]
	root(true)

.mk-welcome:not([data-darkmode])
	root(false)

</style>
