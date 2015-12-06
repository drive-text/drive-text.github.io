# Link tag

	module.exports = (h)->
		css: (href)->
			h 'link',
				href: href
				rel: 'stylesheet'
				type: 'text/css'

		script: (url)->
			h 'script',
				src: url
				type: 'text/javascript'
