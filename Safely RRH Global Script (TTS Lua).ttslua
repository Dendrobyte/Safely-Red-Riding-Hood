--[[

Script by: Mark Bacon (Twitter - @Mobkinz78)

First time scripting in Lua and TTS, so apologies for lack of
proper conventions! Tried to name things as helpfully
as possible, but feel free to reach out if you have any questions.

The start phases are a big buggy, but the host can check roles
in console. You can also shuffle cards in order to choose roles.

]]

-- Set up all the objects --
pathTileID = '74cf28'
trapTileID = 'c4f3df'
choiceTileID = 'a9452b'
safeTileID = 'cecd11'
redRidingHoodID = 'd1ac68'
grandmasHouseID = '3d9987'
blindfoldButtonID = 'de9f7c'
turnTrapTilesButtonID = '63c5cf'
timerID = '15bea2'
startGameID = '7a38f3'

function onLoad()
    -- Name moveable items
    redRidingHoodObject = getObjectFromGUID(redRidingHoodID)
    redRidingHoodObject.setName('Little Red Riding Hood')
    grandmasHouseObject = getObjectFromGUID(grandmasHouseID)
    grandmasHouseObject.setName('Grandmother\'s House')
    blindfoldButtonObject = getObjectFromGUID(blindfoldButtonID)
    blindfoldButtonObject.setName('Blindfold Toggle Button')
    turnTrapTilesButtonObject = getObjectFromGUID(turnTrapTilesButtonID)
    turnTrapTilesButtonObject.setName('Flip trap tiles')
    timerObject = getObjectFromGUID(timerID)
    timerObject.setName('Round Timer Object')
    startGameObject = getObjectFromGUID(startGameID)
    startGameObject.setName('Start Game Button')

    blindfoldButtonObject.createButton({
      click_function = 'toggleBlindfoldsForChoiceTile',
      function_owner = nil,
      label          = 'Choice Tile Timer',
      position       = {0, 1, 0},
      rotation       = {0, 180, 0},
      width          = 1600,
      height         = 1600,
      font_size      = 150,
    })

    turnTrapTilesButtonObject.createButton({
      click_function = 'flipTrapTiles',
      function_owner = nil,
      label          = 'Flip Trap Tiles',
      position       = {0, 1, 0},
      rotation       = {0, 180, 0},
      width          = 1000,
      height         = 1000,
      font_size      = 150,
    })

    startGameObject.createButton({
      click_function = 'onPressStartGame',
      function_owner = nil,
      label          = 'START GAME',
      position       = {0, 1, 0},
      rotation       = {0, 180, 0},
      color          = {0, 10, 5},
      width          = 2000,
      height         = 2000,
      font_size      = 300,
    })

end

-- Button function to begin setup for tiles and initial role assignment --
function onPressStartGame()
  print('Setting up main tile board...')
  setupTileBoard()

  -- Randomly assign roles --
  print('Assigning roles...')
  local runGame = assignRoles()
  if(runGame == 0) then
    return
  end

  print('Starting setup timers...')
  createOneshotTimer('setGreenHouse', 3)
end

-- Function to set up the initial tile board (8x8 board) --
function setupTileBoard()
  -- Create objects --
  pathTileObject = getObjectFromGUID(pathTileID)
  pathTileObject.setName('Game Tile')
  pathTileObject.mass = 400
  trapTileObject = getObjectFromGUID(trapTileID)
  trapTileObject.setName('Game Tile')
  trapTileObject.mass = 400
  choiceTileObject = getObjectFromGUID(choiceTileID)
  choiceTileObject.setName('Game Tile')
  safeTileObject = getObjectFromGUID(safeTileID)
  safeTileObject.setName('Game Tile')

  pathTileTable = {}
  trapTileTable = {}
  choiceTileTable = {}
  safeTileTable = {}

  tileSnappingPoints = {}

  -- Positional counter for object positions to place along columns
  local posCounter = 0
  local xPosAdjuster = 0

  -- Duplicate objects and add to lists for later randomization--
  for i=0,63 do -- Path tiles
    pathTileObjectScale = pathTileObject.getScale()
    local newTilePos = {-10+(2.5*xPosAdjuster), 1, (pathTileObjectScale[3]*(posCounter) + posCounter*1.5)-10}

    newPathTile = pathTileObject.clone({
      position     = newTilePos,
      snap_to_grid = true,
    })
    table.insert(pathTileTable, newPathTile)
    table.insert(tileSnappingPoints, {position = newTilePos, rotation = {0, 0, 0}, rotation_snap = false})

    if posCounter == 7 then
      posCounter = 0
      xPosAdjuster = xPosAdjuster+1
    else
      posCounter = posCounter+1
    end

  end

  for i=0,9 do -- Trap tiles
    local trapTilePos = trapTileObject.getPosition()
    local newTilePos = {trapTilePos[1], trapTilePos[2]+i, trapTilePos[3]}

    table.insert(trapTileTable, trapTileObject.clone({
      position     = newTilePos,
      snap_to_grid = true,
    }))
  end
  trapTileObject.destruct() -- Delete the initial one bc we can

  for i=0,9 do -- Safe tiles
    local safeTilePos = safeTileObject.getPosition()
    local newTilePos = {safeTilePos[1], safeTilePos[2]+i, safeTilePos[3]}

    table.insert(safeTileTable, safeTileObject.clone({
      position     = newTilePos,
      snap_to_grid = true,
    }))
  end
  safeTileObject.destruct() -- Delete the initial one bc we can

  for i=0,9 do -- Choice tiles
    math.randomseed(i*Time.time) -- Make a "truly" random seed
    local randomTileNum = math.random(1, #pathTileTable)

    local choiceTilePos = pathTileTable[randomTileNum].getPosition()
    pathTileTable[randomTileNum].destruct()
    table.remove(pathTileTable, randomTileNum)

    newChoiceTile = choiceTileObject.clone({
      position     = choiceTilePos,
      snap_to_grid = true,
    })

    table.insert(choiceTileTable, newChoiceTile)

  end

  -- Set all snap positions for all tiles --
  for i=1, #pathTileTable do
    pathTileTable[i].setSnapPoints(tileSnappingPoints)
    pathTileTable[i].mass = 10
  end

  for i=1, #trapTileTable do
    trapTileTable[i].setSnapPoints(tileSnappingPoints)
    trapTileTable[i].mass = 10
  end
end

-- Function to randomly assign roles to users --
function assignRoles()
  -- Get all the players
  local playerList = Player.getPlayers()
  if(#playerList < 4) then
    print('There are not enough players to play! You need at least 4')
    return 0
  end

  -- Assign red riding hood
  local randInt = math.random(1, #playerList)
  ridingHood = playerList[randInt].color
  log('ATTENTION! Red Riding Hood is ' .. playerList[randInt].steam_name)
  table.remove(playerList, randInt)

  -- Assign wolf (tell )
  local randInt = math.random(1, #playerList)
  wolfColor = playerList[randInt].color
  log('DEBUG: The wolf is ' .. playerList[randInt].steam_name)
  table.remove(playerList, randInt)
  toggleBlindfolds()
  Players[wolfColor].blindfolded = false

  -- Make the rest of the players grandmothers
  grandmas = {}
  local counter = 1
  for _,playerReference in ipairs(playerList) do
    log('DEBUG: ' .. playerReference.steam_name .. ' is grandma #' .. counter)
    counter = counter+1
    table.insert(grandmas, playerReference)
  end

end

-- Timer methods for starting phases --
function setGreenHouse()
  print('Red riding hood, you have 30 seconds to place grandmother\'s house! Roll the d6 to do so.')

  createOneshotTimer('wolfPlacesTraps', 30)
end

function wolfPlacesTraps()
  print('Wolf, you have 1 minute and 30 seconds to roll the d8 die and place your traps!')

  toggleBlindfolds()
  --Player[wolfColor].blindfolded = false
  print('(Your blind fold has been taken off, 90 seconds to go!)')

  createOneshotTimer('showBoardToGrandmas', 90)
end

function showBoardToGrandmas()
  print('Grandmas, you have 20 seconds to look at the board tiles!')

  toggleBlindfolds()
  --for i=1,#grandmas do
    --grandmas[i].blindfolded = false;
  --end

  flipTrapTiles()
  createOneshotTimer('wrapupSetup', 20)
end

function wrapupSetup()
  flipTrapTiles()

  local playerList = Player.getPlayers()
  for _,playerReference in ipairs(playerList) do
    playerReference.blindfolded = false
  end

  print('Everyone is being shown the board! Red Riding Hood, set your start place and then start the timer!')
end


-- Helper function for creating timers
function createOneshotTimer(fcnName, delay)
    Timer.create({
        identifier = tostring({}), -- unique name
        function_name = fcnName,    -- what it triggers
        function_owner = self,
        delay = delay,              -- delay in seconds
        repetitions = 1,            -- oneshot
    })
end

-- End of timer methods --

function toggleBlindfolds()
  local playerList = Player.getPlayers()
  for _,playerReference in ipairs(playerList) do
    playerReference.blindfolded = true
  end
end

function toggleBlindfoldsForChoiceTile()
  print('Every player except the one chosen by Red Riding Hood please put on your blindfolds!')

  createOneshotTimer('placeChoiceTiles', 5)

end

function placeChoiceTiles()
  print('Chosen one, you have 20 seconds to place your tile!')

  createOneshotTimer('removeBlindfolds', 20)
end

function removeBlindfolds()
  local playerList = Player.getPlayers()
  for _,playerReference in ipairs(playerList) do
    playerReference.blindfolded = false
  end

  print('All blindfolds removed. Resume timer and go!')
end

function flipTrapTiles()
  for i=1,#trapTileTable do
    local currRot = trapTileTable[i].getRotation()
    if(currRot[3] > 150 and currRot[3] < 320) then
      trapTileTable[i].setRotationSmooth({0, 180, 0})
    else
      trapTileTable[i].setRotationSmooth({0, 180, 180})
    end
  end
end

-- Shuffle tables --
function shuffleTable(t)
    for i = #t, 2, -1 do
        local n = math.random(i)
        t[i], t[n] = t[n], t[i]
    end
    return t
end

--[[ The onUpdate event is called once per frame. --]]
function onUpdate()
    --[[ print('onUpdate loop!') --]]
end
