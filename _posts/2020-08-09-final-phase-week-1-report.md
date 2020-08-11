---
layout: post
title: "Final Phase Week 1: Project Comments & Notifications"
author: "Nitish Aggarwal"
categories: journal
---

Hola guys!! We are now into the finale of GSoC Programme. This Week was focussed on improvements in `mobile_app` & implementation of `project_comments` with `fcm_notifications`.

<p align="center">
	<img src="../assets/img/final_countdown.gif" width="300px">
</p>

### Project's Comments

PR [#1590](https://github.com/CircuitVerse/CircuitVerse/pull/1590) aimed at implementing comments for projects. [commontator](https://github.com/lml/commontator/) is being used for handling comments in projects (`acts_as_commontable`). Skimming through the [commontator](https://github.com/lml/commontator/) codebase helped develop a better understanding of how things actually works and implement those as per our requirements..

`comments_serializer` has got several attributes like `has_edit_access` (`@comment.can_be_edited_by?(@current_user)`), `has_delete_access` (`@comment.can_be_deleted_by?(@current_user)`) keeping in mind the accessibility options for the user.

Also `project_serializer` now has various thread attributes including `thread_id` (`@project.commontator_thread.id`) to be able to fetch comments for any `project/commontable`.

### FCM Notifications

Push notifications can be an excellent way to drive user engagement. FCM (Firebase Cloud Messaging) was the service choosen for setting up cross-platform mobile notifications. This will be based on the gem [fcm](https://github.com/spacialdb/fcm) which is kind of a service to hit FCM APIs. Takes in the `fcm_tokens` and queues up the notifications to be sent.

**FCM Token** - A random string to uniquely identify any mobile device notification needs to be sent to.

```ruby
class Fcm < ApplicationRecord
  belongs_to :user

  validates :token, presence: true
  validates :user_id, uniqueness: true
end

class User < ApplicationRecord
  ...
  has_one :fcm, dependent: :destroy
  ...
end
```

Uniqueness validation for `user_id` & not null/empty validation for `token` is handled at database level (`migrations`) too..

The flow that i came up with as follows.. User opens the `mobile_app`, in the `login_viewmodel` or `signup_viewmodel` after successful login/signup, sends `fcm_token` to server through `/api/v1/fcm/token`. Thereafter, `fcm_token` (`@user.fcm.token`) uniquely identifies the device notifs needs to be sent to.. and Voila we have the notif hanging in the tray..

### API Enhancements

PR [#1581](https://github.com/CircuitVerse/CircuitVerse/pull/1581) aims to improve the project's API with the following highlights:

- `public_featured` & `public_users_projects` must be available without being authenticated.
- User's Favourites available at `/api/v1/users/:id/favourites`

Another PR [#1510](https://github.com/CircuitVerse/CircuitVerse/pull/1510) was reviewed & merged (:tada:) which aimed at implementing oauth flow for the API. Token validation was taken care of by fetching `user_info` using `access_tokens` which in turn validates them too :)..

## Next Steps

Currently working on improving on testing suite i.e adding specs for `project_comments` & `fcm_notifications`.. Adding RichText editor for project description in the `mobile_app`. Finally adding tests for `mobile_app`'s viewmodels, interactions and UI would conclude the awesome journey we all have been a part of..
