<template>
<div
	class="note"
	v-show="appearNote.deletedAt == null && !hideThisNote"
	:tabindex="appearNote.deletedAt == null ? '-1' : null"
	:class="{ renote: isRenote, smart: $store.state.device.postStyle == 'smart' }"
	v-hotkey="keymap"
>
	<div class="reply-to" v-if="appearNote.reply && (!$store.getters.isSignedIn || $store.state.settings.showReplyTarget)">
		<x-sub :note="appearNote.reply"/>
	</div>
	<mk-renote class="renote" v-if="isRenote" :note="note" mini/>
	<article>
		<mk-avatar class="avatar" :user="appearNote.user" v-if="$store.state.device.postStyle != 'smart'"/>
		<div class="main">
			<mk-note-header class="header" :note="appearNote" :mini="true"/>
			<div class="body" v-if="appearNote.deletedAt == null">
				<p v-if="appearNote.cw != null" class="cw">
				<misskey-flavored-markdown v-if="appearNote.cw != ''" class="text" :text="appearNote.cw" :author="appearNote.user" :i="$store.state.i" :custom-emojis="appearNote.emojis" />
					<mk-cw-button v-model="showContent"/>
				</p>
				<div class="content" v-show="appearNote.cw == null || showContent">
					<div class="text">
						<span v-if="appearNote.isHidden" style="opacity: 0.5">({{ $t('private') }})</span>
						<a class="reply" v-if="appearNote.reply"><fa icon="reply"/></a>
						<misskey-flavored-markdown v-if="appearNote.text" :text="appearNote.text" :author="appearNote.user" :i="$store.state.i" :custom-emojis="appearNote.emojis"/>
						<a class="rp" v-if="appearNote.renote != null">RN:</a>
					</div>
					<div class="files" v-if="appearNote.files.length > 0">
						<mk-media-list :media-list="appearNote.files"/>
					</div>
					<mk-poll v-if="appearNote.poll" :note="appearNote" ref="pollViewer"/>
					<mk-url-preview v-for="url in urls" :url="url" :key="url"/>
					<a class="location" v-if="appearNote.geo" :href="`https://maps.google.com/maps?q=${appearNote.geo.coordinates[1]},${appearNote.geo.coordinates[0]}`" target="_blank"><fa icon="map-marker-alt"/> {{ $t('location') }}</a>
					<div class="renote" v-if="appearNote.renote"><mk-note-preview :note="appearNote.renote"/></div>
				</div>
				<span class="app" v-if="appearNote.app && $store.state.settings.showVia">via <b>{{ appearNote.app.name }}</b></span>
			</div>
			<footer v-if="appearNote.deletedAt == null">
				<mk-reactions-viewer :note="appearNote" ref="reactionsViewer"/>
				<button @click="reply()">
					<template v-if="appearNote.reply"><fa icon="reply-all"/></template>
					<template v-else><fa icon="reply"/></template>
					<p class="count" v-if="appearNote.repliesCount > 0">{{ appearNote.repliesCount }}</p>
				</button>
				<button @click="renote()" title="Renote">
					<fa icon="retweet"/><p class="count" v-if="appearNote.renoteCount > 0">{{ appearNote.renoteCount }}</p>
				</button>
				<button :class="{ reacted: appearNote.myReaction != null }" @click="react()" ref="reactButton">
					<fa icon="plus"/><p class="count" v-if="appearNote.reactions_count > 0">{{ appearNote.reactions_count }}</p>
				</button>
				<button class="menu" @click="menu()" ref="menuButton">
					<fa icon="ellipsis-h"/>
				</button>
			</footer>
			<div class="deleted" v-if="appearNote.deletedAt != null">{{ $t('deleted') }}</div>
		</div>
	</article>
</div>
</template>

<script lang="ts">
import Vue from 'vue';
import i18n from '../../../i18n';

import XSub from './note.sub.vue';
import noteMixin from '../../../common/scripts/note-mixin';
import noteSubscriber from '../../../common/scripts/note-subscriber';

export default Vue.extend({
	i18n: i18n('mobile/views/components/note.vue'),
	components: {
		XSub
	},

	mixins: [
		noteMixin({
			mobile: true
		}),
		noteSubscriber('note')
	],

	props: {
		note: {
			type: Object,
			required: true
		}
	}
});
</script>

<style lang="stylus" scoped>
.note
	font-size 12px
	border-bottom solid 1px var(--faceDivider)

	&:focus
		z-index 1

		&:after
			content ""
			pointer-events none
			position absolute
			top 2px
			right 2px
			bottom 2px
			left 2px
			border 2px solid var(--primaryAlpha03)
			border-radius 4px

	&:last-of-type
		border-bottom none

	@media (min-width 350px)
		font-size 14px

	@media (min-width 500px)
		font-size 16px

	&.smart
		> article
			> .main
				> header
					align-items center
					margin-bottom 4px

	> .renote + article
		padding-top 8px

	> article
		display flex
		padding 16px 16px 9px

		@media (min-width 600px)
			padding 32px 32px 22px

		> .avatar
			flex-shrink 0
			display block
			margin 0 10px 8px 0
			width 42px
			height 42px
			border-radius 6px
			//position -webkit-sticky
			//position sticky
			//top 62px

			@media (min-width 350px)
				width 48px
				height 48px
				border-radius 6px

			@media (min-width 500px)
				margin-right 16px
				width 58px
				height 58px
				border-radius 8px

		> .main
			flex 1
			min-width 0

			> .header
				@media (min-width 500px)
					margin-bottom 2px

			> .body
				@media (min-width 700px)
					font-size 1.1em

				> .cw
					cursor default
					display block
					margin 0
					padding 0
					overflow-wrap break-word
					color var(--noteText)

					> .text
						margin-right 8px

				> .content

					> .text
						display block
						margin 0
						padding 0
						overflow-wrap break-word
						color var(--noteText)

						> .reply
							margin-right 8px
							color var(--noteText)

						> .rp
							margin-left 4px
							font-style oblique
							color var(--renoteText)

					.mk-url-preview
						margin-top 8px

					> .files
						> img
							display block
							max-width 100%

					> .location
						margin 4px 0
						font-size 12px
						color #ccc

					> .map
						width 100%
						height 200px

						&:empty
							display none

					> .mk-poll
						font-size 80%

					> .renote
						margin 8px 0

						> *
							padding 16px
							border dashed 1px var(--quoteBorder)
							border-radius 8px

				> .app
					font-size 12px
					color #ccc

			> footer
				> button
					margin 0
					padding 8px
					background transparent
					border none
					box-shadow none
					font-size 1em
					color var(--noteActions)
					cursor pointer

					&:not(:last-child)
						margin-right 28px

					&:hover
						color var(--noteActionsHover)

					> .count
						display inline
						margin 0 0 0 8px
						color #999

					&.reacted
						color var(--primary)

			> .deleted
				color var(--noteText)
				opacity 0.7

</style>
