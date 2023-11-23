
A fully featured and deeply tested [Cloudinary](https://cloudinary.com/) [Ghost](https://github.com/TryGhost/Ghost) storage adapter . This works with Node 18

### Features

- Up to date with latest Ghost versions :rocket:
- supports Node v18 



### Install on Docker

Here's an example of using this adapter with a containerized Ghost:

```Dockerfile
FROM ghost:5-alpine as cloudinary
RUN apk add g++ make python3
RUN su-exec node yarn add ghost-storage-cloudinary-mod

FROM ghost:5-alpine
COPY --chown=node:node --from=cloudinary $GHOST_INSTALL/node_modules $GHOST_INSTALL/node_modules
COPY --chown=node:node --from=cloudinary $GHOST_INSTALL/node_modules/ghost-storage-cloudinary-mod $GHOST_INSTALL/content/adapters/storage/ghost-storage-cloudinary-mod
# Here, we use the Ghost CLI to set some pre-defined values.
RUN set -ex; \
    su-exec node ghost config storage.active ghost-storage-cloudinary-mod; \
    su-exec node ghost config storage.ghost-storage-cloudinary-mod.upload.use_filename true; \
    su-exec node ghost config storage.ghost-storage-cloudinary-mod.upload.unique_filename false; \
    su-exec node ghost config storage.ghost-storage-cloudinary-mod.upload.overwrite false; \
    su-exec node ghost config storage.ghost-storage-cloudinary-mod.fetch.quality auto; \
    su-exec node ghost config storage.ghost-storage-cloudinary-mod.fetch.cdn_subdomain true; \
    su-exec node ghost config mail.transport "SMTP"; \
    su-exec node ghost config mail.options.service "Mailgun";
```

Make sure to set the content path right in the Ghost config as well:

```json
"paths": {
    "contentPath": "/var/lib/ghost/content/"
}
```

## Configuration

Check out [configuration.json.dist](configuration.json.dist) for a complete example.

- Ensure to disable Ghost [Image Optimisation](https://ghost.org/docs/concepts/config/#image-optimisation)
- The optional `useDatedFolder = false` to upload images in dated sub-directories (alike default Ghost Local storage adapter)
- The `auth` property is optional if you use the `CLOUDINARY_URL` environment variable [authentification method](https://cloudinary.com/documentation/node_additional_topics#configuration_options)
- The optional `upload` property contains Cloudinary API [upload options](https://cloudinary.com/documentation/image_upload_api_reference#upload)
- The optional `fetch` property contains Cloudinary API [image transformation options](https://cloudinary.com/documentation/image_transformations)

### Recommended configuration

- `upload.use_filename = true` in order use the original image name
- `upload.unique_filename = false` unlikely Ghost local storage adapter which auto-dedup an existing file name, Cloudinary will return the existing image URL instead of deduping the image
- `upload.overwrite = false` goes along with previous option: returns existing image instead of overwriting it
- `upload.folder = "my-blog"` allows to upload all your images into a specific directory instead of Cloudinary media library root
- `upload.tags = ["blog", "photography"]` if you want to add some taxonomy to your uploaded images
- `fetch.quality = "auto"` equals `auto:good` (see [doc](https://cloudinary.com/documentation/image_transformation_reference#quality_parameter))
- `fetch.secure = false` set to true if you want to serve images over SSL (not recommended for performances)
- `fetch.cdn_subdomain = true` to really use the benefit of Cloudinary CDN

:heart: Don't forget to checkout the [plugins](plugins)!

## Development

Run `yarn install` without the `--production` flag.

Runs the tests and generate coverage:

    yarn coverage

Runs the linter:

    yarn eslint

To enable debug logs, set the following environment variable:

    DEBUG=ghost-storage-cloudinary-mod:*

---

Many thanks to @[mmornati](https://github.com/mmornati), @[sethbrasile](https://github.com/sethbrasile) and all other contributors for their work. In the continuation of this project, don't hesitate to fork, contribute and add more features. I'm not the author of this package..
