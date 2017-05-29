---
layout: template1
title: Tutorial - How to make a simple Multiblock Structure
comments: false
---

<p><strong>Minecraft Version:</strong> Tested in 1.6.x and 1.7.x (use getTileEntity instead of getBlockTileEntity in 1.7.x)</p>

<p>MultiBlock structures are a nice way to make advance blocks without dealing with adding multiple recipes as an arbitrary way to make said block expensive. So, how does one go about creating their own MultiBlock? This tutorial will teach you how to make a basic 3x3x3 MultiBlock structure with a hollow center using a Master-Slave system.</p>

<p>This tutorial assumes you have basic knowledge of how to make a basic mod, creating TileEntities, and dealing with NBT data. If you don’t, I recommend reading/watching Wuppy’s tutorials, they’re an excellent source of information and are also the tutorials I used when I first started modding. ;)</p>

<p>Please note that there are multiple ways to create a MultiBlock structure and this method may not be the best. Also note that the way we will be making the MultiBlock will make it like the ones in RailCraft, in which all the data is stored in the bottom center block of the structure, so keep that in mind if you don’t want your structure to work that way.</p>

<p><strong>Step 1: Create a basic TileEntity</strong></p>

<p>To get started, where going to need a basic TileEntity, which I will be calling TileMultiBlock for this tutorial. I won’t go over what each method is for, but just note this will be our starting point.</p>

<p>We will also be needing a few variables, two boolean for the tile to see if it has a master and/or if it is the master of the structure and three integers for the coordinates of the master for the slaves to use. Make sure to add getters and setters for each one and have them saved in the Tile’s NBT data.</p>

<p><a class="btn btn-danger btn-default" onclick="toggle_visibility('code0');">Show/Hide Code</a></p>

<div id="code0" style="display:none;">
	<pre><code class="language-java">
package net.lomeli.tutorial;

import net.minecraft.nbt.NBTTagCompound;
import net.minecraft.nbt.NBTTagList;
import net.minecraft.tileentity.TileEntity;

public class TileMultiBlock extends TileEntity {
    private boolean hasMaster, isMaster;
    private int masterX, masterY, masterZ;

    @Override
    public void updateEntity() {
    }

    @Override
    public void writeToNBT(NBTTagCompound data) {
        super.writeToNBT(data);
        data.setInteger("masterX", masterX);
        data.setInteger("masterY", masterY);
        data.setInteger("masterZ", masterZ);
        data.setBoolean("hasMaster", hasMaster);
        data.setBoolean("isMaster", isMaster);
        if (hasMaster() &amp;&amp; isMaster()) {
            // Any other values should ONLY BE SAVED TO THE MASTER
        }
    }

    @Override
    public void readFromNBT(NBTTagCompound data) {
        super.readFromNBT(data);
        masterX = data.getInteger("masterX");
        masterY = data.getInteger("masterY");
        masterZ = data.getInteger("masterZ");
        hasMaster = data.getBoolean("hasMaster");
        isMaster = data.getBoolean("isMaster");
        if (hasMaster() &amp;&amp; isMaster()) {
            // Any other values should ONLY BE READ BY THE MASTER
        }
    }

    public boolean hasMaster() {
        return hasMaster;
    }

    public boolean isMaster() {
        return isMaster;
    }

    public int getMasterX() {
        return masterX;
    }

    public int getMasterY() {
        return masterY;
    }

    public int getMasterZ() {
        return masterZ;
    }

    public void setHasMaster(boolean bool) {
        hasMaster = bool;
    }

    public void setIsMaster(boolean bool) {
        isMaster = bool;
    }

    public void setMasterCoords(int x, int y, int z) {
        masterX = x;
        masterY = y;
        masterZ = z;
    }
}
</code></pre>
</div>

<p><strong>Step 2: Check and form MultiBlock</strong></p>

<p>Next step is to make a method to check is the structure was properly formed. The easiest way is to make three <em>for loops</em> (for the x, y, and z coordinates) and check to see if there is a TileEntity at each coordinate and counting it if it is an instance of TileMultiBlock. Then counting them up to see if the right number of TileEntities are present and checking if the center is empty.</p>

<p><a class="btn btn-danger btn-default" onclick="toggle_visibility('code1');">Show/Hide Code</a></p>

<div id="code1" style="display:none;">
	<pre><code class="language-java">
public boolean checkMultiBlockForm() {
    int i = 0;
    // Scan a 3x3x3 area, starting with the bottom left corner
    for (int x = xCoord - 1; x &lt; xCoord + 2; x++)
        for (int y = yCoord; y &lt; yCoord + 3; y++)
            for (int z = zCoord - 1; z &lt; zCoord + 2; z++) {
                 TileEntity tile = worldObj.getBlockTileEntity(x, y, z);
                 // Make sure tile isn't null, is an instance of the same Tile, and isn't already a part of a multiblock
                 if (tile != null &amp;&amp; (tile instanceof TileMultiBlock)) {
                     if (this.isMaster()) {
                         if (((TileMultiBlock)tile).hasMaster())
                             i++;
                     } else if (!((TileMultiBlock)tile).hasMaster())
                         i++;
                 }
             }
     // check if there are 26 blocks present ((3*3*3) - 1) and check that center block is empty
     return i &gt; 25 &amp;&amp; worldObj.isAirBlock(xCoord, yCoord + 1, zCoord);
}
</code></pre>
</div>

<p>We also need a method to tell all those tiles that they now have a master and where it’s located at. You’ll also have to tell the bottom center block that it is the master. This method will only be used by the master once it detects that the structure is formed.</p>

<p><a class="btn btn-danger btn-default" onclick="toggle_visibility('code2');">Show/Hide Code</a></p>

<div id="code2" style="display:none;">
	<pre><code class="language-java">
public void setupStructure() {
    for (int x = xCoord - 1; x &lt; xCoord + 2; x++)
        for (int y = yCoord; y &lt; yCoord + 3; y++)
            for (int z = zCoord - 1; z &lt; zCoord + 2; z++) {
                TileEntity tile = worldObj.getBlockTileEntity(x, y, z);
                // Check if block is bottom center block
                boolean master = (x == xCoord &amp;&amp; y == yCoord &amp;&amp; z == zCoord);
                if (tile != null &amp;&amp; (tile instanceof TileMultiBlock)) {
                    ((TileMultiBlock) tile).setMasterCoords(xCoord, yCoord, zCoord);
                    ((TileMultiBlock) tile).setHasMaster(true);
                    ((TileMultiBlock) tile).setIsMaster(master);
                }
            }
}
</code></pre>
</div>

<p><strong>Step 3: Resetting the blocks</strong></p>

<p>Now we need setup methods to check if the structure is invalidated and to reset all the parts. The master block can reuse the <em>checkMultiBlockForm</em> method for checking the structure, but the slave blocks should also constantly check that the master still exists in case the master is broken first, and for them to reset themselves if it no longer exists.</p>

<p><a class="btn btn-danger btn-default" onclick="toggle_visibility('code3');">Show/Hide Code</a></p>

<div id="code3" style="display:none;">
	<pre><code class="language-java">
// Reset method to be run when the master is gone or tells them to
public void reset() {
    masterX = 0;
    masterY = 0;
    masterZ = 0;
    hasMaster = false;
    isMaster = false;
}

public boolean checkForMaster() {
    TileEntity tile = worldObj.getBlockTileEntity(masterX, masterY, masterZ);
    return (tile != null &amp;&amp; (tile instanceof TileMultiBlock));
}
</code></pre>
</div>

<p>We also need a method for the master to use if <em>checkMultiBlockForm</em> returns false. Since we already have the reset method, we can just make a clone of <em>setupStructure</em> that resets stuff instead.</p>

<p><a class="btn btn-danger btn-default" onclick="toggle_visibility('code4');">Show/Hide Code</a></p>

<div id="code4" style="display:none;">
	<pre><code class="language-java">
public void resetStructure() {
    for (int x = xCoord - 1; x &lt; xCoord + 2; x++)
        for (int y = yCoord; y &lt; yCoord + 3; y++)
            for (int z = zCoord - 1; z &lt; zCoord + 2; z++) {
                TileEntity tile = worldObj.getBlockTileEntity(x, y, z);
                if (tile != null &amp;&amp; (tile instanceof TileMultiBlock))
                    ((TileMultiBlock) tile).reset();
            }
}
</code></pre>
</div>

<p><strong>Step 4: Putting it all together</strong></p>

<p>Now that we have all the parts we need ready, all that’s left is to make it actually run. What we basically need is for the each block to constantly check if the structure is formed until it is. Once it is, we only want it to check when there is a block change, in which the master will check if the structure is formed and the slaves check if the master still exists.</p>

<p>First, lets make sure our tiles check for the structure is formed and to make sure only the master block does work. All of this will be put in the <em>updateEntity</em> method.</p>

<p><a class="btn btn-danger btn-default" onclick="toggle_visibility('code5');">Show/Hide Code</a></p>

<div id="code5" style="display:none;">
	<pre><code class="language-java">
@Override
public void updateEntity() {
    super.updateEntity();
    if (!worldObj.isRemote) {
        if (hasMaster()) { 
            if (isMaster()) {
                // Put stuff you want the multiblock to do here!
            }
        } else {
            // Constantly check if structure is formed until it is.
            if (checkMultiBlockForm())
                setupStructure();
        }
    }
}
</code></pre>
</div>

<p>Now, we need to have our master check if the structure is still formed or our slaves to check if the master is still in tact, which we will do in our onNeighborBlockChange in our block class.</p>

<p><a class="btn btn-danger btn-default" onclick="toggle_visibility('code6');">Show/Hide Code</a></p>

<div id="code6" style="display:none;">
	<pre><code class="language-java">
@Override
public void onNeighborBlockChange(World world, int x, int y, int z, Block block) {
    TileEntity tile = world.getTileEntity(x, y, z);
    if (tile != null &amp;&amp; tile instanceof TileMultiBlock) {
        TileMultiBlock multiBlock = (TileMultiBlock) tile;
        if (multiBlock.hasMaster()) {
            if (multiBlock.isMaster()) {
                if (!multiBlock.checkMultiBlockForm())
                    multiBlock.resetStructure();
            } else {
                if (!multiBlock.checkForMaster())
                    multiBlock.reset();
            }
        }
    }
    super.onNeighborBlockChange(world, x, y, z, block);
}
</code></pre>
</div>

<p><strong>Final Code</strong></p>

<p><a class="btn btn-danger btn-default" onclick="toggle_visibility('code7');">TileEntity Code</a></p>

<div id="code7" style="display:none;">
	<pre><code class="language-java">
package net.lomeli.tutorial;
 
import net.minecraft.nbt.NBTTagCompound;
import net.minecraft.nbt.NBTTagList;
import net.minecraft.tileentity.TileEntity;
 
public class TileMultiBlock extends TileEntity {
    private boolean hasMaster, isMaster;
    private int masterX, masterY, masterZ;
 
    @Override
    public void updateEntity() {
        super.updateEntity();
        if (!worldObj.isRemote) {
            if (hasMaster()) { 
                if (isMaster()) {
                    // Put stuff you want the multiblock to do here!
                }
            } else {
                // Constantly check if structure is formed until it is.
                if (checkMultiBlockForm())
                    setupStructure();
            }
        }
    }
    
    /** Check that structure is properly formed */
	public boolean checkMultiBlockForm() {
		int i = 0;
		// Scan a 3x3x3 area, starting with the bottom left corner
		for (int x = xCoord - 1; x &lt; xCoord + 2; x++)
			for (int y = yCoord; y &lt; yCoord + 3; y++)
				for (int z = zCoord - 1; z &lt; zCoord + 2; z++) {
					TileEntity tile = worldObj.getBlockTileEntity(x, y, z);
					// Make sure tile isn't null, is an instance of the same Tile, and isn't already a part of a multiblock
					if (tile != null &amp;&amp; (tile instanceof TileMultiBlock)) {
						if (this.isMaster()) {
							if (((TileMultiBlock)tile).hasMaster())
								i++;
						} else if (!((TileMultiBlock)tile).hasMaster())
							i++;
					}
			}
		// check if there are 26 blocks present ((3*3*3) - 1) and check that center block is empty
		return i &gt; 25 &amp;&amp; worldObj.isAirBlock(xCoord, yCoord + 1, zCoord);
	}
 
    /** Setup all the blocks in the structure*/
    public void setupStructure() {
        for (int x = xCoord - 1; x &lt; xCoord + 2; x++)
            for (int y = yCoord; y &lt; yCoord + 3; y++)
                for (int z = zCoord - 1; z &lt; zCoord + 2; z++) {
                    TileEntity tile = worldObj.getBlockTileEntity(x, y, z);
                    // Check if block is bottom center block
                    boolean master = (x == xCoord &amp;&amp; y == yCoord &amp;&amp; z == zCoord);
                    if (tile != null &amp;&amp; (tile instanceof TileMultiBlock)) {
                        ((TileMultiBlock) tile).setMasterCoords(xCoord, yCoord, zCoord);
                        ((TileMultiBlock) tile).setHasMaster(true);
                        ((TileMultiBlock) tile).setIsMaster(master);
                    }
                }
    }
 
    /** Reset method to be run when the master is gone or tells them to */
    public void reset() {
        masterX = 0;
        masterY = 0;
        masterZ = 0;
        hasMaster = false;
        isMaster = false;
    }
 
    /** Check that the master exists */
    public boolean checkForMaster() {
        TileEntity tile = worldObj.getBlockTileEntity(masterX, masterY, masterZ);
        return (tile != null &amp;&amp; (tile instanceof TileMultiBlock));
    }
    
    /** Reset all the parts of the structure */
    public void resetStructure() {
        for (int x = xCoord - 1; x &lt; xCoord + 2; x++)
            for (int y = yCoord; y &lt; yCoord + 3; y++)
                for (int z = zCoord - 1; z &lt; zCoord + 2; z++) {
                    TileEntity tile = worldObj.getBlockTileEntity(x, y, z);
                    if (tile != null &amp;&amp; (tile instanceof TileMultiBlock))
                        ((TileMultiBlock) tile).reset();
                }
    }
 
    @Override
    public void writeToNBT(NBTTagCompound data) {
        super.writeToNBT(data);
        data.setInteger("masterX", masterX);
        data.setInteger("masterY", masterY);
        data.setInteger("masterZ", masterZ);
        data.setBoolean("hasMaster", hasMaster);
        data.setBoolean("isMaster", isMaster);
        if (hasMaster() &amp;&amp; isMaster()) {
            // Any other values should ONLY BE SAVED TO THE MASTER
        }
    }

    @Override
    public void readFromNBT(NBTTagCompound data) {
        super.readFromNBT(data);
        masterX = data.getInteger("masterX");
        masterY = data.getInteger("masterY");
        masterZ = data.getInteger("masterZ");
        hasMaster = data.getBoolean("hasMaster");
        isMaster = data.getBoolean("isMaster");
        if (hasMaster() &amp;&amp; isMaster()) {
            // Any other values should ONLY BE READ BY THE MASTER
        }
    }
 
    public boolean hasMaster() {
        return hasMaster;
    }
 
    public boolean isMaster() {
        return isMaster;
    }
 
    public int getMasterX() {
        return masterX;
    }
 
    public int getMasterY() {
        return masterY;
    }
 
    public int getMasterZ() {
        return masterZ;
    }
 
    public void setHasMaster(boolean bool) {
        hasMaster = bool;
    }
 
    public void setIsMaster(boolean bool) {
        isMaster = bool;
    }
 
    public void setMasterCoords(int x, int y, int z) {
        masterX = x;
        masterY = y;
        masterZ = z;
    }
}
</code></pre>
</div>

<p><a class="btn btn-danger btn-default" onclick="toggle_visibility('code8');">Block Code</a></p>

<div id="code8" style="display:none;">
	<pre><code class="language-java">
package net.lomeli.tutorial;

import net.minecraft.block.Block;
import net.minecraft.block.BlockContainer;
import net.minecraft.block.material.Material;
import net.minecraft.tileentity.TileEntity;
import net.minecraft.world.World;

public class BlockMultiBlock extends BlockContainer {
    public BlockMultiBlock(Material material) {
        super(material);
    }

    @Override
    public void onNeighborBlockChange(World world, int x, int y, int z, Block block) {
        TileEntity tile = world.getBlockTileEntity(x, y, z);
        if (tile != null &amp;&amp; tile instanceof TileMultiBlock) {
            TileMultiBlock multiBlock = (TileMultiBlock) tile;
            if (multiBlock.hasMaster()) {
                if (multiBlock.isMaster()) {
                    if (!multiBlock.checkMultiBlockForm())
                        multiBlock.resetStructure();
                } else {
                    if (!multiBlock.checkForMaster()) {
                        multiBlock.reset();
                        world.markBlockForUpdate(x, y, z);
                    }
                }
            }
        }
        super.onNeighborBlockChange(world, x, y, z, block);
    }

    @Override
    public TileEntity createNewTileEntity(World world, int meta) {
        return TileMultiBlock();
    }
}
</code></pre>
</div>

<p>And there ya go. Not too difficult when you look at it. Again, probably not the best way to do it, but it works. Remember that the master should be the only one doing work and saving NBT data, so remember that when you writing your code (I added comments for locations where it should be placed for a basic tile entity). If I’m missing anything, or if you don’t understand, leave a comment or contact me.</p>

<p>Since a few people have been having problems understanding this without an example, I made an example mod that is based of this tutorial. You can take a look for yourself here: <a href="https://github.com/Lomeli12/MultiBlock-Tutorial/" target="_blank">https://github.com/Lomeli12/MultiBlock-Tutorial/</a></p>

<p>Edit: Small edit to take into account if a Tile is already a part of a mulitblock structure.<br />
	Edit 2: Fixed typos and missing TileEntity objects.<br />
	Edit 3: Fixed some things that may break the multiblock.<br />
	Edit 4: Thanks to <a href="https://twitter.com/KitsuneKihira" target="_blank">Kihira</a> for pointing out it’s I should only re-validate the structure on block neighbour change.<br />
	Edit 5: Moved my website to a new server and cms. Using Prism for highlighting.</p>

<p>Edit 6: Found a nice replacement for the code hightlighting/show hide =D</p>
