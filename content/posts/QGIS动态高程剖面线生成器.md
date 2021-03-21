---
title: "动态高程轮廓线[QGIS符号系统几何生成器]"
author: "iPattern"
date: 2019-09-23T02:23:00+08:00
description: "This is meta description"
categories: ["编程 Programming"]
tags: ["GIS","Visualization"]
---

**注意**：需要QGIS最新版或等待QGIS 3.10正式版。我是个潮人。

在QGIS中加载栅格图层。

（在任意坐标系统中）添加一个新的多边形`Scratch`图层。将其`符号系统`设置为`Inverted Polygons`。使用几何生成器`Geometry Generator`作为符号图层类型。将其设置为`LineString / MultiLineString`。在下面输入表达式并修改为当前`图层ID`（最好进入表达式编辑器，然后在“图层”树中找到它）。然后在最底部调整比例因子。

 ![img](https://tva1.sinaimg.cn/large/006y8mN6gy1g78v0yjc8ij30qj0nagms.jpg)
 ![img](https://tva1.sinaimg.cn/large/006y8mN6gy1g78v1c314mj30qj0naqio.jpg)
 ![img](https://tva1.sinaimg.cn/large/006y8mN6gy1g78v1h8ns7j30qj0nagu8.jpg)
 ![img](https://tva1.sinaimg.cn/large/006y8mN6gy1g78uur80mlj30qj0na4kk.jpg)
 ![img](https://tva1.sinaimg.cn/large/006y8mN6gy1g78uxm7u54j30qj0naduy.jpg)
 ![img](https://tva1.sinaimg.cn/large/006y8mN6gy1g78uxnjsp6j30qj0na3zw.jpg)
 ![img](https://tva1.sinaimg.cn/large/006y8mN6gy1g78uxolo5fj30qj0naq6n.jpg)
 ![img](https://tva1.sinaimg.cn/large/006y8mN6gy1g78uxp1fu8j30qj0na76f.jpg)
 ![img](https://tva1.sinaimg.cn/large/006y8mN6gy1g78uxqw1qyj30qj0nak8n.jpg)
 ![img](https://tva1.sinaimg.cn/large/006y8mN6gy1g78uxvyjzaj30qj0nadm0.jpg)
 ![img](https://tva1.sinaimg.cn/large/006y8mN6gy1g78uxrvatoj30qj0nak61.jpg)


```-- UPPER CASE comments below are where you can change things
-- UPPER CASE comments below are where you can change things

with_variable(
	'raster_layer',
	'long_and_complicated_layer_id',  -- RASTER LAYER to sample from
	-- this collects all the linestrings generated below into one multilinestring
	collect_geometries(
		-- a loop for each y value of the grid
		array_foreach(
			-- array_foreach loops over all elements of the series generated below
			-- which is a range of numbers from the bottom to the top of y values
			-- of the map canvas extent coordinates.
			-- the result will be an array of linestrings
			generate_series(
				y(@map_extent_center)-(@map_extent_height/2), -- bottom y
				y(@map_extent_center)+(@map_extent_height/2),  -- top y
				@map_extent_height/50  -- stepsize -> HOW MANY LINES
			),
			
			-- we want to enter another loop so we assign the name 'y' to
			-- the current element of the array_foreach loop
			with_variable(
				'y',
				@element,
				
				-- now we are ready to generate the line for this y value
				make_line(
					-- another loop, this time for the x values. same logic as before
					-- the result will be an array of points
					array_foreach(
						generate_series(
							x(@map_extent_center)-(@map_extent_width/2), -- left x
							x(@map_extent_center)+(@map_extent_width/2),  -- right x
							@map_extent_width/50  -- stepsize -> HOW MANY POINTS PER LINE
						),
						-- and here we create each point of the line
						make_point(
							@element,  -- the current value from the loop over the x value range
							@y  -- the y value from the outer loop
							+   -- will get an additional offset to generate the effect
							-- we look for values at _this point_ in the raster, and since
							-- the raster might not have any value here, we must use coalesce
							-- to use a replacement value in those cases
							coalesce(  -- coalesce to catch raster null values
								raster_value(
									@raster_layer,
									1,  -- band 1, *snore*
									-- to look up the raster value we need to look in the right position
									-- so we make a sampling point in the same CRS as the raster layer
									transform( 
										make_point(@element, @y),
										@map_crs,
										layer_property(@raster_layer,'crs')
									)
								),
								0  -- coalesce 0 if raster_value gave null
							-- here is where we set the scaling factor for the raster -> y values
							-- if things are weird, set it to 0 and try small multiplications or divisions
							-- to see what happens.
							-- for metric systems you will want to multiply
							-- for geographic coordinates you will want to divide
							)*10  -- user-defined factor for VERTICAL EXAGGERATION
						)
					)
				)
			)
		)
	)
)  -- wee
```

如果你的栅格数据不是保存在固态硬盘`SSD`上，这个操作可能比较慢。

当然，==修改坐标系统也不影响它正常工作==！



###### This entry was posted in [cartography](https://hannes.enjoys.it/blog/category/cartography/), [qgis](https://hannes.enjoys.it/blog/category/qgis/) on [2019-09-16](https://hannes.enjoys.it/blog/2019/09/dynamic-elevation-profile-lines-as-qgis-geometry-generator/).

###### https://hannes.enjoys.it/blog/2019/09/dynamic-elevation-profile-lines-as-qgis-geometry-generator/