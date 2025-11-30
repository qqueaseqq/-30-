public class FunTimeAttackAuraRotation {
    
    private final Minecraft mc = Minecraft.getMinecraft();
    private long lastAttackTime = 0;
    private final long attackDelay = 55;
    private int tickCounter = 0;
    
    public void onTick() {
        if (shouldAttack()) {
            executeRotation();
        }
    }
    
    private boolean shouldAttack() {
        Entity target = getBestTarget();
        return target != null && mc.thePlayer.getDistanceToEntity(target) <= 3.5;
    }
    
    private void executeRotation() {
        long currentTime = System.currentTimeMillis();
        
        if (currentTime - lastAttackTime < attackDelay) {
            return;
        }
        
        Entity target = getBestTarget();
        if (target == null) return;
        
        rotateToTarget(target);
        
        long attackDelayVariation = calculateOptimalDelay(target);
        
        if (currentTime - lastAttackTime >= attackDelayVariation) {
            mc.thePlayer.swingItem();
            mc.playerController.attackEntity(mc.thePlayer, target);
            lastAttackTime = currentTime;
        }
    }
    
    private long calculateOptimalDelay(Entity target) {
        tickCounter++;
        
        if (target.hurtResistantTime > 10) {
            return 70 + (target.hurtResistantTime % 20) * 2;
        }
        
        if (tickCounter % 20 < 10) {
            return 55;
        } else {
            return 65;
        }
    }
    
    private void rotateToTarget(Entity target) {
        float[] rotations = getRotationsToTarget(target);
        
        float yaw = rotations[0];
        float pitch = rotations[1];
        
        float currentYaw = mc.thePlayer.rotationYaw;
        float currentPitch = mc.thePlayer.rotationPitch;
        
        float yawDiff = MathHelper.wrapAngleTo180_float(yaw - currentYaw);
        float pitchDiff = MathHelper.wrapAngleTo180_float(pitch - currentPitch);
        
        float maxYawChange = 25.0f;
        float maxPitchChange = 15.0f;
        
        if (Math.abs(yawDiff) > maxYawChange) {
            yawDiff = yawDiff > 0 ? maxYawChange : -maxYawChange;
        }
        if (Math.abs(pitchDiff) > maxPitchChange) {
            pitchDiff = pitchDiff > 0 ? maxPitchChange : -maxPitchChange;
        }
        
        mc.thePlayer.rotationYaw = currentYaw + yawDiff;
        mc.thePlayer.rotationPitch = currentPitch + pitchDiff;
    }
    
    private float[] getRotationsToTarget(Entity target) {
        double deltaX = target.posX - mc.thePlayer.posX;
        double deltaY = target.posY + target.getEyeHeight() * 0.9 - (mc.thePlayer.posY + mc.thePlayer.getEyeHeight());
        double deltaZ = target.posZ - mc.thePlayer.posZ;
        
        double distance = Math.sqrt(deltaX * deltaX + deltaZ * deltaZ);
        
        float yaw = (float) (Math.atan2(deltaZ, deltaX) * 180.0 / Math.PI) - 90.0f;
        float pitch = (float) -(Math.atan2(deltaY, distance) * 180.0 / Math.PI);
        
        return new float[]{yaw, pitch};
    }
    
    private Entity getBestTarget() {
        Entity bestTarget = null;
        double bestDistance = 3.5;
        
        for (Entity entity : mc.theWorld.loadedEntityList) {
            if (!(entity instanceof EntityLivingBase) || entity == mc.thePlayer) {
                continue;
            }
            
            EntityLivingBase livingEntity = (EntityLivingBase) entity;
            
            if (!livingEntity.isEntityAlive() || livingEntity.deathTime > 0) {
                continue;
            }
            
            double distance = mc.thePlayer.getDistanceToEntity(livingEntity);
            if (distance <= bestDistance && isValidTarget(livingEntity)) {
                bestDistance = distance;
                bestTarget = livingEntity;
            }
        }
        
        return bestTarget;
    }
    
    private boolean isValidTarget(EntityLivingBase entity) {
        return entity != mc.thePlayer && 
               entity.isEntityAlive() && 
               !entity.isInvisible() && 
               entity.getDistanceToEntity(mc.thePlayer) <= 3.5;
    }
}
