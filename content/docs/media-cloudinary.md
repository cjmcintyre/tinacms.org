---
title: Using Media with Cloudinary
---

Manage **Cloudinary media assets** in TinaCMS.

## Installation

```bash
yarn add next-tinacms-cloudinary @tinacms/auth
```

## Connect with Cloudinary

You need some credentials provided by Cloudinary to set this up properly. If you do not already have an account, you can ([register here](https://cloudinary.com/users/register/free).

**next-tinacms-cloudinary** uses environment variables within the context of a Next.js site to properly access your Cloudinary account.

Add the following variables to an `.env` file.

```
NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME=<Your Cloudinary Cloud Name>
NEXT_PUBLIC_CLOUDINARY_API_KEY=<Your Cloudinary API key>
CLOUDINARY_API_SECRET=<Your Cloudinary API secret>
```

## Register the Media Store

Now, you can register the Cloudinary Media store with the instance of Tina in your app by passing the `TinaCloudCloudinaryMediaStore` to the `TinaCMS` instance via its `mediaStore` prop.

This is also where we can update our `mediaOptions` on the cms object.

```tsx
import { TinaEditProvider } from "tinacms/dist/edit-state";
import { TinaCloudCloudinaryMediaStore } from 'next-tinacms-cloudinary'

const TinaCMS = dynamic(() => import("tinacms"), { ssr: false });

const App = ({ Component, pageProps }) => {
  return (
    <>
      <TinaEditProvider
        editMode={
          <TinaCMS
            ...
            mediaStore={TinaCloudCloudinaryMediaStore}
            {...pageProps}
          >
         ...
         </TinaCMS>
        }
      >
      ...
```

## Set up API routes

Set up a new API route in the `pages` directory of your Next.js app, e.g. `pages/api/cloudinary`.
Then add a new catch all API route for media.

Call `createMediaHandler` to set up routes and connect your instance of the Media Store to your Cloudinary account.

Import `isAuthorized` from [`tina-cloud-next`](https://github.com/tinacms/tinacms/tree/main/packages/tina-cloud-next).

The `authorized` key will make it so only authorized users within Tina Cloud can upload and make media edits.

```ts
//[...media].ts

import {
  mediaHandlerConfig,
  createMediaHandler,
} from 'next-tinacms-cloudinary/dist/handlers'

import { isAuthorized } from '@tinacms/auth'

export const config = mediaHandlerConfig

export default createMediaHandler({
  cloud_name: process.env.NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME,
  api_key: process.env.NEXT_PUBLIC_CLOUDINARY_API_KEY,
  api_secret: process.env.CLOUDINARY_API_SECRET,
  authorized: async (req, _res) => {
    try {
      const user = await isAuthorized(req)

      return user && user.verified
    } catch (e) {
      console.error(e)
      return false
    }
  },
})
```

## Update Schema

Now that the media store is registered and the API route for media set up, let's add an image to your schema.

In your `.tina/schema.ts` add a new field for the image, e.g:

```ts
 {
  name: 'hero',
  type: 'image',
  label: 'Hero Image',
 }
```

Now, when editing your site, the image field will allow you to connect to your Cloudinary account via the Media Store to manage your media assets.