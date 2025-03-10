<template>
	<div class="KdPost" :class="{'KdPost--flipped':  flipped}">
		<div class="KdPost__timestamp Timestamp" v-if="timestamped">
			{{timestamp}}
			<div class="Timestamp__dot"></div>
		</div>

		<div class="KdPost__header">
			<div class="KdPost__title-time">
				<h3 class="KdPost__title">
					<kd-link :href="post.url">{{post.title}}</kd-link>
				</h3>
				<time class="KdPost__time" :datetime="datetime">{{datetext}}</time>
			</div>
			<span class="KdPost__readtime">{{readtime}}</span>
			<div class="KdPost__tags">
				<kd-tag v-if="post.primary_tag" :tag="post.primary_tag" />
			</div>
		</div>

		<kd-link class="KdPost__image-link" :href="post.url">
			<img class="KdPost__image" v-if="post.feature_image" :src="post.feature_image">
		</kd-link>

		<div class="KdPost__content">
			<p class="KdPost__excerpt"
				:class="{'KdPost__excerpt--noncustom': !post.custom_excerpt}"
				v-html="excerpt">
			</p>
		</div>

		<div class="KdPost__footer">
			<kd-continue class="KdPost__continue" :href="post.url" />
		</div>
	</div>
</template>

<style lang="less" scoped>
	@import "../less/utils.less";

	.KdPost {
		background: rgba(var(--grey-900), 1);
		transition: all .5s ease;
		transform-origin: top center;
		transform: rotateX(-90deg);
		opacity: 0;

		&--flipped {
			transform: rotateX(0deg);
			opacity: 1;
		}

		&__header {
			color: rgba(var(--grey-050), 1);
			display: flex;
			flex-direction: column;
			justify-content: space-between;
			padding: 25px 30px;
		}

		&__title-time {
			display: flex;
		}

		&__title {
			width: 0;
			flex: 1;
			margin: 0;
			margin-right: 10px;
			color: rgba(var(--grey-050), 1);
			font-family: var(--font-sans);
			font-weight: 700;
			font-size: 1.3rem;
			letter-spacing: -0.02em;
			white-space: nowrap;
			overflow: hidden;
			text-overflow: ellipsis;
			text-decoration: none;
			user-select: none;

			* {
				color: inherit;
				text-decoration: inherit;
			}
		}

		&__readtime {
			display: block;
			font-family: var(--font-sans);
			font-size: .8rem;
			margin-bottom: 10px;
			user-select: none;
		}

		&__time {
			font-family: var(--font-sans);
			font-weight: 200;
			font-size: 1.2rem;
			white-space: nowrap;
			user-select: none;
		}

		&__tags {
			display: flex;
		}

		&__image-link {
			user-select: none;
		}

		&__image {
			display: inline-block;
			width: 100%;
			height: 200px;
			object-fit: cover;
		}

		&__content {
			color: rgba(var(--grey-050), 1);
			padding: 15px 30px;
			user-select: none;
		}

		&__excerpt {
			position: relative;
			margin: 0;
			font-size: .97rem;
			letter-spacing: -0.015em;
			max-height: 8rem;
			line-height: 1.6rem;
			text-align: justify;
			overflow: hidden;
			word-break: break-all;
			font-family: var(--font-sans);

			&--noncustom::after {
				.OverflowFade;
			}
		}

		&__footer {
			padding-top: 0;
			padding-left: 30px;
			padding-bottom: 30px;
		}
	}

	.Timestamp {
		position: sticky;
		top: 100px;
		height: 0;
		transform: translate(-10rem, 30px);
		color: rgba(var(--grey-100), 1);
		font-family: var(--font-ui);
		font-weight: 400;
		font-size: 1.1rem;

		&__dot {
			display: inline-block;
			width: 7px;
			height: 7px;
			margin-left: 10px;
			background: rgba(var(--grey-100), 1);
			border-radius: 50%;
			vertical-align: middle;
		}
	}

	@media (max-width: 1500px) {
		.KdPost {
			&__time {
				font-size: 1rem;
			}
		}
	}

	@media (max-width: 1300px) {
		.KdPost {
			&__time {
				font-size: 1.2rem;
			}
		}
	}

	@media (max-width: 768px) {
		.KdPost {
			&__time {
				font-size: 1rem;
			}
		}
	}
</style>

<i18n>
{
	"ko": {
		"readtime": "읽는데 {time}"
	},

	"en": {
		"readtime": "{time} to read"
	},

	"ja": {
		"readtime": "{time}で読める"
	}
}
</i18n>

<script>
	import calculateReadtime from "@/src/calculateReadtime";
	import createExcerpt from "@/src/createExcerpt";
	import dateLocale, { getLocale } from "@/src/dateLocale";
	import { format, formatDistanceToNow, isSameWeek, isToday } from "date-fns";

	import KdContinue from "@/components/KdContinue";
	import KdLink from "@/components/KdLink";
	import KdTag from "@/components/KdTag";

	export default {
		data() {
			return {
				flipped: false
			};
		},

		props: {
			post: {
				type: Object,
				required: true
			},

			index: {
				type: Number
			},

			timestamped: {
				type:  Boolean
			}
		},

		computed: {
			readtime() {
				return this.$t('readtime', {
					time: calculateReadtime(this.post)
				});
			},

			datetime() {
				return this.post.published_at;
			},

			datetext() {
				const current = new Date();
				const date = new Date(this.post.published_at);

				if(!isSameWeek(current, date)) {
					return format(date, 'PPP', dateLocale());
				}

				if(!isToday(date)) {
					return format(date, 'EEEE', dateLocale());
				}

				return formatDistanceToNow(date, { addSuffix: true, locale: getLocale() });
			},

			timestamp() {
				return format(new Date(this.post.published_at), 'yyyy. MM. dd', dateLocale());
			},

			htmlExcerpt() {
				return createExcerpt(this.post.html);
			},

			excerpt() {
				if(this.post.custom_excerpt)
					return this.post.custom_excerpt;

				return this.htmlExcerpt;
			}
		},

		mounted() {
			setTimeout(() => { this.flipped = true }, this.index / 2 * 500);
		},

		components: {
			KdContinue,
			KdLink,
			KdTag
		}
	};
</script>
