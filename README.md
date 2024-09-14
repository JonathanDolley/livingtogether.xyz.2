# hugo-toha.github.io

[![Netlify Status](https://api.netlify.com/api/v1/badges/b1b93b02-f278-440b-ae1b-304e9f4c4ab5/deploy-status)](https://app.netlify.com/sites/toha/deploys) [![Build Status](https://img.shields.io/endpoint.svg?url=https%3A%2F%2Factions-badge.atrox.dev%2Fhugo-toha%2Fhugo-toha.github.io%2Fbadge%3Fref%3Dmain&style=flat)](https://actions-badge.atrox.dev/hugo-toha/hugo-toha.github.io/goto?ref=main) ![Repository Size](https://img.shields.io/github/repo-size/hugo-toha/hugo-toha.github.io) ![Contributor](https://img.shields.io/github/contributors/hugo-toha/hugo-toha.github.io) ![Last Commit](https://img.shields.io/github/last-commit/hugo-toha/hugo-toha.github.io) ![License](https://img.shields.io/github/license/hugo-toha/hugo-toha.github.io) ![Open Issues](https://img.shields.io/github/issues/hugo-toha/hugo-toha.github.io?color=important) ![Open Pull Requests](https://img.shields.io/github/issues-pr/hugo-toha/hugo-toha.github.io?color=yellowgreen) ![Security Headers](https://img.shields.io/security-headers?url=https%3A%2F%2Fhugo-toha.github.io%2F) [![This project is using Percy.io for visual regression testing.](https://percy.io/static/images/percy-badge.svg)](https://percy.io/b7cb60ab/hugo-toha.github.io)

An example hugo static site with Toha theme.

Attributions:
- <a href='https://www.freepik.com/vectors/business'>Business vector created by studiogstock - www.freepik.com</a>

Logo created using: https://www.brandcrowd.com/maker/mylogos/subscriptions

# Image shortcode (imgh)

Thanks to: https://www.brycewray.com/posts/2022/06/responsive-optimized-images-hugo/

for this shortcode for better image processing.

## imgh.html

{{- $respSizes := slice "320" "640" "960" "1280" "1600" "1920" -}}
{{/*
	These are breakpoints, in pixels.
	Adjust these to fit your use cases.
	Obviously, the more breakpoints,
	the more images you'll be producing.
	(Fortunately, Hugo does that
	**really** fast, as you'd expect,
	but watch out for any storage
	issues this can present either
	locally or in your online repo,
	especially if you have a really
	large number of original images.)
*/}}
{{- $imgBase := "images/" -}}
{{/*
	This will be from top-level `assets/images`,
	where we'll keep all images for Hugo's
	processing (this makes them "global
	resources," as noted in the documentation).
*/}}
{{- $src := resources.Get (printf "%s%s" $imgBase (.Get "src")) -}}
{{- $alt := .Get "alt" -}}
{{- $divClass := "" -}}{{/* Init'g */}}
{{/*
	The styling in $imgClass, below, makes
	an image fill the container horizontally
	and adjust its height automatically
	for that, and then fade in for the LQIP effect.
	Feel free to adjust your CSS/SCSS as desired.
*/}}
{{- $imgClass := "w-full h-auto animate-fade" -}}
{{- $dataSzes := "(min-width: 1024px) 100vw, 50vw" -}}
{{/*
	Now we'll create the 20-pixel-wide LQIP
	and turn it into Base64-encoded data, which
	is better for performance and caching.
*/}}
{{- $LQIP_img := $src.Resize "20x jpg" -}}
{{- $LQIP_b64 := $LQIP_img.Content | base64Encode -}}
{{/*
	$CFPstyle is for use in styling
	the div's background, as you'll see shortly.
*/}}
{{- $CFPstyle := printf "%s%s%s" "background: url(data:image/jpeg;base64," $LQIP_b64 "); background-size: cover; background-repeat: no-repeat;" -}}
{{/*
	Then, we create a 640-pixel-wide JPG
	of the image. This will serve as the
	"fallback" image for that tiny percentage
	of browsers that don't understand the
	HTML `picture` tag.
*/}}
{{- $actualImg := $src.Resize "640x jpg" -}}
{{/*
	Now we'll handle the LQIP background for the
	div that will contain the image content; the
	conditional at the top controls whether we're
	doing inline styling --- which is a no-no for
	a tight Content Security Policy (CSP). Here,
	it checks whether we're using nonces (and thus
	a tight CSP), as spec'd in the site config file.
	If so, it creates a new class, named
	with an md5 hash for the value of $src, that
	the div can use to provide the LQIP background.
	Otherwise, it inserts inline styling.
	**THEREFORE** . . .
	If you don't have a problem with inline styling,
	feel free to use only the second option and
	avoid the conditional altogether.
*/}}
{{- $imgBd5 := md5 $src -}}
{{- if .Site.Params.Nonces -}}
	<style>
		.imgB-{{ $imgBd5 }} { {{ $CFPstyle | safeCSS }} }
	</style>
	<div class="relative imgB-{{ $imgBd5 }} bg-center">
{{- else -}}
	<div class="relative bg-center" style="{{ $CFPstyle | safeCSS }}">
{{- end -}}
{{/*
	Now we'll build the `picture` which modern
	browsers use to decide which image, and
	which format thereof, to show. Remember to
	put `webp` first, since the browser will use
	the first format it **can** use, and WebP files
	usually are smaller. After WebP, the fallback
	is the universally safe JPG format.
*/}}
	<picture>
		<source
			type="image/webp"
			srcset="
			{{- with $respSizes -}}
				{{- range $i, $e := . -}}
					{{- if ge $src.Width . -}}
						{{- if $i }}, {{ end -}}{{- ($src.Resize (printf "%sx%s" . " webp") ).RelPermalink }} {{ . }}w
					{{- end -}}
				{{- end -}}
			{{- end -}}"
			sizes="{{ $dataSzes }}"
		/>
		<source
			type="image/jpeg"
			srcset="
			{{- with $respSizes -}}
				{{- range $i, $e := . -}}
					{{- if ge $src.Width . -}}
						{{- if $i }}, {{ end -}}{{- ($src.Resize (printf "%sx%s" . " jpg") ).RelPermalink }} {{ . }}w
					{{- end -}}
				{{- end -}}
			{{- end -}}"
			sizes="{{ $dataSzes }}"
		/>
		<img class="{{ $imgClass }}"
			src="{{ $actualImg.RelPermalink }}"
			width="{{ $src.Width }}"
			height="{{ $src.Height }}"
			alt="{{ $alt }}"
			loading="lazy"
		/>
	</picture>
</div>

## Calling the imgh shortcode

{{< imgh src="my-pet-cat_3264x2448.jpg" alt="Photo of a cat named Shakespeare sitting on a window sill" >}}
