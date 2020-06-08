---
layout: post
title: "Week 4: How about Grading?"
author: "Nitish Aggarwal"
categories: journal
---

This week i have been working on adding assignment grading APIs functionality. Yeah i know, students will now on be evaluated for their submissions..
_student screams_

<p align="center">
	<img src="../assets/img/mr_bean_grades.gif">
</p>

Obviously `grades_controller.rb` would be handling creating, updating and deleting grades. There was some existing controller logic for grading but was not going well with the REST principles.. So, tweaked them a little bit for the good :)

```ruby
# POST /api/v1/assignments/:assignment_id/projects/:project_id/grade
def create
  @grade = Grade.new(
    assignment_id: params[:assignment_id], project_id: params[:project_id]
  )
  @grade.user_id = @current_user.id
  @grade.grade = grade_params[:grade]
  @grade.remarks = sanitize grade_params[:remarks]

  if @grade.save
    render json: Api::V1::GradeSerializer.new(@grade), status: :created
  else
    api_error(status: 422, errors: @grade.errors)
  end
end

# PATCH /api/v1/grades/:id
def update
  @grade.update!(grade_params)
  render json: Api::V1::GradeSerializer.new(@grade), status: :accepted
end

# DELETE /api/v1/grades/:id
def destroy
  @grade.destroy!
  render json: {}, status: :no_content
end
```

Grading API details can be referenced in [#1482](https://github.com/CircuitVerse/CircuitVerse/pull/1482). While testing the above mentioned API, tried hitting the `CREATE` API twice and two grades with same `project_id` & `assignment_id` were created, weird right? This took me straight over to `rails c` to test if this was actually the case and yes we found a vulnerability that was until now handled in frontend only..

So, there were different approaches for the uniqueness check but adding index & model level validation won!! obviously with some feedback from my mentor..

I was excited for this because this was my first time adding some migration.. So, it all begins with `rails generate migration AddUniqueGradeValidation` which generated a empty migration, added composite `:index` to `grades` with `unique: true`.

```ruby
class AddUniqueGradeValidation < ActiveRecord::Migration[6.0]
  def change
    add_index :grades, %i[project_id assignment_id], unique: true
  end
end
```

Model level validation was basically [uniqueness](https://guides.rubyonrails.org/active_record_validations.html#uniqueness) validation of `:project_id` scoped with `assignment_id`. Here's is a glimpse..

```ruby
class Grade < ApplicationRecord
  ...
  validates :project_id, uniqueness: { scope: :assignment_id }

end
```

This is how we roll.. The one of many times i was happy when presented with errors :p.. Some enhancements to projects API were also made in view of the `mobile_app`, serializing `author_name` & `selective including` to name a few.. For more details head over to [#1490](https://github.com/CircuitVerse/CircuitVerse/pull/1490).

With this we have the basic API developed, for the complete documentation of the API you gotta go to [docs](https://nitish145.github.io/slate/).

<p align="center" >
	<img src="../assets/img/we_did_it.gif" width="300px">
</p>

_That's all folks :)_

Some time was also devoted perfecting my previous work [#1462](https://github.com/CircuitVerse/CircuitVerse/pull/1462) and [#1473](https://github.com/CircuitVerse/CircuitVerse/pull/1473). We now have a single codeclimate complexity issue only.. _smirks_

Next up I'll be working on the `mobile_app`, the second part of my GSoC project, stay tuned for the amazing stuff and..

> I'll be back
