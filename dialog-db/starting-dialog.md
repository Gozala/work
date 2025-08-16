---
title: Dialog - Personal Substrate for Cooperative Computing
date: 2025-08-12
---

# Dialog - Personal Substrate for Cooperative Computing

You're planning dinner for friends this weekend. Your calendar knows who's coming. Your contacts know Sarah's gluten-free, Tom's vegetarian. As you open a meal planner, it suggests recipes everyone can enjoy, portions adjusted for your group. Your shopping list populates with needed ingredients.

No configurations. No authorizations. No information retrieval from corporate silos. Your tools cooperate through a shared substrate—each deriving insights and contributing new understanding. They complement each other, becoming true [bicycles for your mind][].

## Means of Cooperation

Today's tools cooperate through following coordination models:

**Corporate suites** (Google Workspace, Microsoft Office) work together because one company controls everything.

**Limited integrations** where companies choose which APIs to expose. Slack decides who to integrate with, not you. You can't message Signal friends from WeChat.

**Manual coordination** for everything else—you copy data between tools, maintain consistency, context-switch endlessly.

Your contacts live in one app, dietary preferences in another, calendar events in a third. You are the integration layer.

## The Paradigm Shift

Large Language Models enable anyone to generate specialized tools. A food blogger creates a meal planner. A fitness coach builds a nutrition tracker. Each speaks its own language.

This renders existing cooperation models completely inadequate:
- No corporation controls thousands of indie tools (hopefully)
- Too many tools for each one to integrate with the rest
- Manual coordination becomes impossible

When data fragments across origins, cooperation requires concerted effort from all parties. Will these tools trap your data in new silos? Or enable emergent cooperation on a substrate you control?

## Dialog: The Solution

Dialog is a knowledge substrate—your personal context that tools derive insights from and contribute back to. Built on semantic facts, everything becomes queryable:

```javascript
{ the: "contact/name", of: sarah, is: "Sarah Chen" }
{ the: "dietary/restriction", of: sarah, is: "gluten-free" }
{ the: "event/guest", of: dinner_saturday, is: sarah }
```

Meal planners find dietary restrictions of dinner guests. Shopping lists find ingredients and group size. No brittle integrations—just shared facts enabling emergent cooperation.

Like Git, Dialog supports branching, forking, and selective collaboration. Every fact assertion is auditable and reversible—you can see which tool added what and when. Local yet ubiquitous through encrypted sync. Switch providers anytime. Your data, your rules.

## The Technical Approach

Dialog synthesizes proven research:
- **Semantic facts** (from Datomic) eliminate schema migrations
- **[Probabilistic Search Trees]** enable efficient partial sync without downloading entire datasets
- **Datalog queries** enable powerful pattern matching and inference
- **Commodity blob storage** reduces cloud to encrypted, dumb pipes

Queries run on your device. Remote storage never sees unencrypted data. Tools see only what you allow them to. Switching tools means switching lenses, not migrating databases.

## Why Now

AI will spawn thousands of specialized tools. Current coordination models can't scale to this future. The technical pieces exist—we just need implementation.

We can build more walled gardens and hope for the best. Or we can build personal substrate for cooperative computing.

The PC revolution put computational power in users' hands. The next revolution puts computational cooperation under users' control.

---

*If this vision resonates—if you see seamless tool cooperation as inevitable and necessary—I want to hear from you. Whether you're a developer tired of silos, a funder who sees the shift coming, or someone who believes software should serve its users, let's talk.*

[bicycles for your mind]: https://www.youtube.com/watch?v=KmuP8gsgWb8
[Probabilistic Search Trees]:https://g-trees.github.io/g_trees/
