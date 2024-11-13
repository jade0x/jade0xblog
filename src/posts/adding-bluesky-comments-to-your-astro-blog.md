---
layout: ../layouts/BlogPost.astro
title: Adding Bluesky Comments to Your Astro Blog
slug: Adding-Bluesky-Comments-to-Your-Astro-Blog
description: Adding Bluesky Comments to Your Astro Blog (Using Cassidoo's Template)
tags:
  - technical
added: 2024-11-13T06:00:00.000Z
blueskyUri: 'at://did:plc:f5qcuwzgwrwe6oxfygf33pud/app.bsky.feed.post/3latn7jee222c'
---

My blog now has Bluesky comments!

A few months ago I came across [Emily's post about using Bluesky replies as a comment section](https://bsky.app/profile/emilyliu.me/post/3kveqmh5v5v2o), where she implemented [Samuel's method of integrating Bluesky's open protocol as a comment system.](https://graysky.app/blog/2024-02-05-adding-blog-comments)

I love the idea, but I use [Cassidoo's Astro blog template](https://github.com/cassidoo/blahg) (and TinaCMS), so I needed to make some adjustments.

**Credit Where Credit's Due**

* [Samuel Newman](https://bsky.app/profile/did:plc:p2cp5gopk7mgjegy6wadk3ep) for the original write-up on using Bluesky's open protocol for blog comments
* [Emily Liu](https://bsky.app/profile/emilyliu.me) for sharing her implementation
* [Cassidy Williams](https://bsky.app/profile/cassidoo.co) (Cassidoo) for the excellent Astro blog template

## Guide

If you're using Cassidoo's template with TinaCMS, here's how to add Bluesky comments to your blog:

### 1. Update TinaCMS Configuration

First, add a field for the Bluesky URI in your TinaCMS configuration. In your tina/config.js file, add this to your fields array:

```javascript
{
  type: "string",
    name: "blueskyUri",
      label: "Bluesky Post URI",
        description: "URI of the corresponding Bluesky post for comments",
}
```

### 2. Create the Comments Component

Create a new file called CommentSection.astro in your src/components directory with this code:

```javascript
---
  interface Props {
  uri: string;
}

interface BlueskyPost {
  post: {
    author: {
      avatar: string;
      displayName: string;
      handle: string;
    };
    record: {
      text: string;
    };
    indexedAt: string;
    likeCount: number;
    repostCount: number;
    replyCount: number;
  };
  replies?: BlueskyPost[];
}

const { uri } = Astro.props;

// Get the post ID for the Bluesky link
const postId = uri.split('/').pop();
const blueskyLink = `https://bsky.app/profile/${uri.split('/')[2]}/post/${postId}`;

let comments: BlueskyPost[] = [];
let error = null;

if (uri) {
  try {
    const endpoint = `https://api.bsky.app/xrpc/app.bsky.feed.getPostThread?uri=${encodeURIComponent(uri)}`;

    const response = await fetch(endpoint, {
      method: 'GET',
      headers: {
        'Accept': 'application/json'
      }
    });

    if (!response.ok) {
      throw new Error(`Failed to fetch comments: ${response.status}`);
    }

    const data = await response.json();
    if (data.thread?.replies) {
      comments = data.thread.replies;
    }
  } catch (e) {
    error = e.message;
    console.error('Error:', e);
  }
}
---

<div class="comments-section">
  <h2>Comments</h2>
  <p class="comment-prompt">
    Reply on Bluesky <a href={blueskyLink} target="_blank" rel="noopener noreferrer">here</a> to join the conversation.
  </p>
  
  {error && <p class="error">Error loading comments: {error}</p>}
  
  {!error && comments.length === 0 && (
    <p class="no-comments">No comments yet. Be the first to comment!</p>
  )}

  {!error && comments.length > 0 && (
    <div class="comments-list">
      {comments.map((comment) => (
        <div class="comment">
          <div class="comment-header">
            <img src={comment.post.author.avatar} alt={`${comment.post.author.displayName}'s avatar`} class="avatar" />
            <div class="author-info">
              <span class="author-name">{comment.post.author.displayName}</span>
              <span class="author-handle">@{comment.post.author.handle}</span>
            </div>
          </div>
          <div class="comment-content">
            {comment.post.record.text}
          </div>
          <div class="comment-footer">
            <div class="interaction-counts">
              <span>{comment.post.replyCount || 0} 💬</span>
              <span>{comment.post.repostCount || 0} 🔁</span>
              <span>{comment.post.likeCount || 0} ❤️</span>
            </div>
            <time datetime={comment.post.indexedAt}>
              {new Date(comment.post.indexedAt).toLocaleDateString()}
            </time>
          </div>
          {comment.replies && comment.replies.length > 0 && (
            <div class="replies">
              {comment.replies.map((reply) => (
                <div class="reply comment">
                  <div class="comment-header">
                    <img src={reply.post.author.avatar} alt={`${reply.post.author.displayName}'s avatar`} class="avatar" />
                    <div class="author-info">
                      <span class="author-name">{reply.post.author.displayName}</span>
                      <span class="author-handle">@{reply.post.author.handle}</span>
                    </div>
                  </div>
                  <div class="comment-content">
                    {reply.post.record.text}
                  </div>
                  <div class="comment-footer">
                    <div class="interaction-counts">
                      <span>{reply.post.replyCount || 0} 💬</span>
                      <span>{reply.post.repostCount || 0} 🔁</span>
                      <span>{reply.post.likeCount || 0} ❤️</span>
                    </div>
                    <time datetime={reply.post.indexedAt}>
                      {new Date(reply.post.indexedAt).toLocaleDateString()}
                    </time>
                  </div>
                </div>
              ))}
            </div>
          )}
        </div>
      ))}
    </div>
  )}
</div>

<style>
  /* Basic styling - users can customize this */
  .comments-section {
    margin-top: 2rem;
    padding-top: 1rem;
  }

  .comment {
    margin: 1rem 0;
    padding: 1rem;
  }

  .comment-header {
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }

  .avatar {
    width: 32px;
    height: 32px;
    border-radius: 50%;
  }

  .author-info {
    display: flex;
    flex-direction: column;
  }

  .comment-footer {
    display: flex;
    justify-content: space-between;
    margin-top: 0.5rem;
  }

  .interaction-counts {
    display: flex;
    gap: 1rem;
  }

  .replies {
    margin-left: 1.5rem;
    padding-left: 1rem;
  }
</style>
```

### 3. Update Your Blog Post Layout

In your blog post layout file (likely src/layouts/BlogPost.astro), add the new component:

```javascript
---
import CommentSection from "../components/CommentSection.astro";
// ... other imports
---

< !--Add this where you want comments to appear-- >
  { content.blueskyUri && <CommentSection uri={content.blueskyUri} /> }
```

## Using the Feature

Each time you create a blog post:

1. Write and publish your blog post
2. Share it on Bluesky
3. Get the Bluesky post's URI:
   * Go to your Bluesky post
   * Click the timestamp to open the single post view
4. Format the URI as: at://\[your-DID]/app.bsky.feed.post/\[post-id]
5. Add this URI to your blog post's "Bluesky Post URI" field in TinaCMS
6. Publish your changes

Your blog will now display any Bluesky replies to your post as comments!

Feel free to customize the styling to match your blog's aesthetic, and feel free to share your blog and let me know what you think 😀
