---
layout: post
title: "Week 3: Let's Collaborate"
author: "Nitish Aggarwal"
categories: journal
---

Hola peeps, This week was primarily focussed on writing specs for Groups, members and assignments endpoints while adding collaborators for the projects. This Week started with the user's endpoints being live [here](https://circuitverse.org/api/v1/users).. kinda feels happy to see your work go live :)

<p align="center">
	<img src="../assets/img/happy_minions.gif">
</p>

Let's start with `collaborators_controller.rb` which would handle listing, creating and deleting collaborators. I had an initial thought of creating `collaboration_serializer` but with some constructive feedback loops set up with my mentor [@tachyons](https://github.com/tachyons/), soon realized it being an overkill, hence switched to `collaborators` instead of `collaboration`.

```ruby
class Project < ApplicationRecord
  has_many :collaborators, type: user
end
```

`collaborator` being a `user` only also didn't called for a separate `collaborator_serializer`. Some naming tweaks to the exisiting `user_serializer` and we were good to go..

```ruby
# /api/v1/projects/:project_id/collaborators
def index
  @collaborators = paginate(@project.collaborators)
  # only name needs to be serialized for a collaborator
  @options = {
    params: { only_name: true },
    links: link_attrs(@collaborators, api_v1_project_collaborators_url)
  }
  render json: Api::V1::UserSerializer.new(@collaborators, @options)
end

# POST /api/v1/projects/:project_id/collaborators
def create
  parse_mails_except_current_user(params[:emails])
  newly_added.each do |email|
    user = User.find_by(email: email)
    if user.present?
      # create collaboration
      Collaboration.where(project_id: @project.id, user_id: user.id).first_or_create
    end
  end
end

# DELETE /api/v1//projects/:project_id/collaborators/:id
# :id is essentially the user_id for the user to be removed from project
def destroy
  # we would want to delete the associated collaboration not the collaborator obviously
  @collaboration = Collaboration.find_by(user: @collaborator, project: @project)
  @collaboration.destroy!

  render json: {}, status: :no_content
end
```

You can head over to [#1473](https://github.com/CircuitVerse/CircuitVerse/pull/1473) for more details regarding collaborations API if you would want to. This week we had some improvements in the previously added Groups & Assignments API too, also following some improvements in Projects API [#1441](https://github.com/CircuitVerse/CircuitVerse/pull/1441) got merged :)

Some configurational changes to `.codeclimate.yml` best suited for specs were also made in view of [#1441](https://github.com/CircuitVerse/CircuitVerse/pull/1441), also worked upon improving and updating the [**docs**](https://nitish145.github.io/slate/).

_That's all folks :)_

Some improvements needs to be done in [#1462](https://github.com/CircuitVerse/CircuitVerse/pull/1462) and [#1473](https://github.com/CircuitVerse/CircuitVerse/pull/1473) to be given a green signal. Next up we also have FCM notifications lined, stay tuned for the updates..

> See Ya Later
