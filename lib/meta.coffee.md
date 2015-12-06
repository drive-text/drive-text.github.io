# Meta tags

	module.exports = (h)->
		author: (content)->
			h 'meta',
				content: content
				name: 'author'

		charset: (content='utf-8')->
			h 'meta', charset: content

		description: (content)->
			h 'meta',
				content: content
				name: 'description'

		http_equiv: (content='IE=edge', http_equiv='X-UA-Compatible')->
			h 'meta',
				content: content
				'http-equiv': http_equiv

		viewport: (content="width=device-width, initial-scale=1")->
			h 'meta',
				content: content
				name: 'viewport'
