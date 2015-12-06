# Base template

## Imports

	fs = require 'fs'

	h = require 'hyperscript'

	link = require('./lib/link')(h)

	meta = require('./lib/meta')(h)


## Classes

	hc = (sel, classes, args...)->
		classes_text = classes.join '.'

		h "#{sel}.#{classes_text}", args...


## Get page content

	module.exports = (h)->
		h 'html', lang: 'en',
			h 'head', [
				meta.charset()

				meta.author 'Matt McKellar-Spence'

				meta.description 'Editor for text files in Google Drive, such as CSV, JSON, Markdown or source code.'

				meta.http_equiv()

				meta.viewport()

				link.css 'https://storage.googleapis.com/code.getmdl.io/1.0.6/material.indigo-pink.min.css'

				link.script 'https://storage.googleapis.com/code.getmdl.io/1.0.6/material.min.js'

				h 'title', 'Google Drive text editor.'
			]

			h 'body', [
				h '#loader_wrapper.mdl-grid', [
					hc 'div#progress_bar', [
						'mdl-progress'
						'mdl-js-progress'
						'mdl-progress__indeterminate'
						'mdl-cell'
						'mdl-cell--12-col'
					]
				]

				hc 'div#editor_wrapper', [
					'mdl-layout'
					'mdl-layout--fixed-header'
					'mdl-js-layout'
					'mdl-color--grey-100'
					'hidden'
				], [
					hc 'header', [
						'mdl-layout__header'
						'mdl-layout__header--scroll'
						'mdl-color--grey-100'
						'mdl-color-text--grey-800'
					], [
						h '.mdl-layout__header-row', [
							h 'span.mdl-layout-title', 'Google Drive text editor'

							h '.mdl-layout-spacer'

							hc 'button#save_button', [
								'mdl-button'
								'mdl-js-button'
							], 'Save'
						]
					]

					h 'main.mdl-layout__content', [
						h '.mdl-grid', [
							hc 'div', [
								'mdl-color--white'
								'mdl-shadow--4dp'
								'content'
								'mdl-color-text--grey-800'
								'mdl-cell'
								'mdl-cell--12-col'
							], [
								hc 'div', [
									'mdl-textfield'
									'mdl-js-textfield'
									'mdl-cell'
									'mdl-cell--12-col'
								], [
									h 'textarea#file_content.mdl-textfield__input',
										rows: 24
										type: 'text'
								]
							]
						]
					]
				]

				link.script '/index.js'

				link.script 'https://apis.google.com/js/client.js?onload=google_client_onload'
			]


## Build page

	content = module.exports(h).outerHTML

	fs.writeFileSync 'index.html', content, 'utf8'
