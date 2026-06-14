I'm using index instanced draw calls to draw objects. With I'm indexing into a buffer device address buffer, to do an gl_InstanceIndex for a position of an object.

This works fine, the problem is how I set my `vkCmdDrawIndexed();`. I have an array of models, these arrays simply store how many instances I have of that model. I have an entities array that stores an index into the model array. 

`models[entities[0].modelIndex].countOfModels; 

So in this example entity `0` might have an `modelIndex` of `99` which has `1000` instances of that model.

Now I'm looping through all my model instances like this.

```c++
for (uint32_t modelIndexType = 0; modelIndexType < models.size; ++modelIndexType)
{

	vkCmdPushConstants(vkCommandBuffer, vkPipelineLayout, VK_SHADER_STAGE_VERTEX_BIT, 0, sizeof(pushConstants), &pushConstants);
	
	vkCmdBindIndexBuffer(vkCommandBuffer, vkBuffer, offset, indexType);

	if (modelIndexType == models.size - 1)
	{
		vkCmdDrawIndexed(vkCommandBuffer, meshDraw.count, models[modelIndexType].countOfModelType, 0, 0, 0);
	}
	else
	{
		vkCmdDrawIndexed(vkCommandBuffer, meshDraw.count, models[modelIndexType].countOfModelType, 0, 0, models[modelIndexType + 1].countOfModelType);
	}
}
```

This is were things go wrong. When I delete an entity from `entities` and the entity data from `entityData` and correspond to an instance. In the example below I'm deleting entity `0`

```c++
if(deleted)
{
	uint32_t element = 0;

	if (entities.size != 0)
	{
		uint32_t modelIndex = scene.entities[element].modelIndex;

		scene.models[modelIndex].countOfModelType -= 1;
	}

	if (entities.size != 0)
	{
		entities.deleteSwap(element);
		entityData.deleteSwap(element);
	}
}
```

The BDA I use to instance over was created with the `entityData` array. I use a delete swap on this array and on the `entities` so there no reference to `models` index anymore.



